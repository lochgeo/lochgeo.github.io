---
layout: post
title:  "Correlation IDs - Following One Request Through the Noise"
date:   2020-12-01 18:41:00
comments: True
categories: [Software, Observability]
excerpt_separator: "<!--more-->"
---

Last Tuesday I got a Slack ping at 9:40pm: "payments are slow, can you look?" I opened Kibana, typed `status:500`, and watched a waterfall of red. Five services. Three pods each. Everyone logging something useful. Nobody logging the *same* something. Twenty minutes later I still could not answer the only question that mattered — which user action started this mess?

That is the night I stopped treating correlation IDs as a nice-to-have and started treating them like a seatbelt.

<!--more-->

### What a correlation ID actually is

It is not magic. It is a single opaque string — usually a GUID — created when a request enters your system, then stamped on every log line, every outbound HTTP call, and every message you put on a queue for that same unit of work.

When it works, you type one filter in Seq or Kibana and you get the full story: gateway → order API → inventory → payment → reply. When it does not work, you get five half-stories and a war room.

I think of it like a baggage tag at the airport. The suitcase changes conveyor belts and planes. The tag stays the same. Without the tag you are just yelling "black roller bag!" into a hangar.

### What ASP.NET Core already gives you

On .NET Core 3.1 (still what most of our services run, even with .NET 5 out a few weeks ago), you already get pieces of this for free:

- **`HttpContext.TraceIdentifier`** — unique per request on that process. Fine inside one API. Useless across services unless you *forward* something.
- **`System.Diagnostics.Activity`** — the distributed-tracing primitive. ASP.NET Core and `HttpClient` know how to start activities and pass IDs on outbound calls.
- **Log scopes** — Microsoft.Extensions.Logging will attach `RequestId`, and when an Activity is present, `TraceId` / `SpanId` show up on log lines.

That last bit is the one people miss. If you wire Serilog (or the built-in logger) correctly and an Activity is flowing, your structured logs already carry a trace identifier. You do not need a second custom field *if* every hop speaks the same protocol.

### Hierarchical vs W3C — pick one and stick to it

Through .NET Core 3.x the default Activity ID format is the older hierarchical `Request-Id` header style (`|root.child.`). W3C Trace Context — the `traceparent` header — is supported, but you opt in:

```csharp
public static void Main(string[] args)
{
    Activity.DefaultIdFormat = ActivityIdFormat.W3C;
    Activity.ForceDefaultIdFormat = true;
    CreateHostBuilder(args).Build().Run();
}
```

I set this in every new service now. Heterogeneous stacks (Node gateway, Java worker, .NET API) are happier when everyone speaks `traceparent`. Staying on hierarchical forever is fine *inside* a pure .NET estate; mixing formats without thinking is how you get two IDs for one request and trust neither.

Microsoft's own guidance from the .NET Core 3 timeframe is basically: prefer W3C going forward, keep hierarchical only for compatibility with older peers.

### The boring middleware that saves your evening

Even with Activities, teams still want a simple `X-Correlation-ID` (or similar) that frontends and support tools can copy-paste. The pattern I keep shipping:

1. On inbound request: read `X-Correlation-ID` if present; otherwise generate a GUID (or use `Activity.Current?.TraceId`).
2. Push it into Serilog's `LogContext` for the request lifetime.
3. Echo it on the response so the SPA / mobile client can show "ref: abc-123" on an error screen.
4. On outbound `HttpClient` calls: send the same header (or rely on Activity propagation if you have standardized on W3C).
5. On queue messages: put the ID in message headers/metadata, not buried in the JSON body only one consumer knows about.

Sketch of the LogContext bit:

```csharp
public async Task InvokeAsync(HttpContext context)
{
    var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault()
        ?? Activity.Current?.TraceId.ToString()
        ?? Guid.NewGuid().ToString("N");

    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        context.Response.OnStarting(() =>
        {
            context.Response.Headers["X-Correlation-ID"] = correlationId;
            return Task.CompletedTask;
        });

        await _next(context);
    }
}
```

Register it early. After routing is fine; after a random auth middleware that already logged three lines without the ID is not.

### Structured logs or you are still grepping

A correlation ID on a free-text line like `"error for correlation abc"` is better than nothing and worse than a property. With Serilog message templates you want:

```csharp
Log.Warning("Payment declined for {OrderId} with {ReasonCode}", orderId, reason);
// CorrelationId comes from LogContext — do not paste it into the message string
```

Then in Seq you filter `CorrelationId = '...'` and optionally `OrderId = 42`. That is the whole point of structured logging: properties you can query, not novels you can skim.

Enrich every service with at least `Application` / `Service` name too. Five microservices dumping into one index without a service property is how you recreate the original problem with extra steps.

### Where it usually breaks

A few failure modes I have hit this year, all while sitting on the same dining chair that became my office:

- **Gateway generates an ID; internal service generates another.** Someone "helpfully" always creates a new GUID and ignores the inbound header.
- **HTTP works; the bus does not.** Background consumers start fresh with no ID. The user-facing request looks fine; the failed retry at 2am is an orphan.
- **Fire-and-forget `Task.Run` without capturing context.** The continuation logs, but LogContext / Activity did not flow. Async is not optional knowledge here.
- **PII in baggage.** Stuffing user email into a propagated header because "support needs it" is a compliance conversation waiting to happen. Keep the ID opaque; look up the user in a system that is allowed to know.
- **Trusting client-supplied IDs on public endpoints.** Accepting an inbound correlation header from the open internet is fine for *correlation*; do not use it as a security boundary or a cache key.

### What I want on every service checklist

Before I call a service "observable enough" for on-call:

- One well-known correlation / trace ID on every request log line
- That ID returned to the caller on errors
- Outbound HTTP and queue messages forward it
- Structured sink (Seq, Elasticsearch, whatever you run) with the ID as a first-class field
- A runbook line: "paste this ID into the log UI"

Distributed tracing products and OpenTelemetry are maturing fast, and I am glad they are. But correlation IDs are the 80% fix you can finish on a Friday afternoon without a platform rewrite. If your logs cannot answer "show me everything for this one user click," the fancy dashboard is decoration.

Next time payments are slow at 9:40pm, I want the first query — not the twentieth — to be the one that works.
