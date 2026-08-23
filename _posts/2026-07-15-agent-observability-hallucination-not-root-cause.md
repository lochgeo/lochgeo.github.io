---
layout: post
title:  "Agent Observability - 'It Hallucinated' Is Not a Root Cause"
date:   2026-07-15 09:41:00
comments: True
categories: [Software, Observability]
excerpt_separator: "<!--more-->"
---

Last month one of our internal chatbots gave a confidently wrong answer about a collateral margin rule, and the incident thread quietly died with the words "looks like a hallucination, closing as won't fix". I have started hearing that phrase the way I used to hear "it's flaky". Technically true, completely useless, and usually a polite way of saying nobody looked.

Six years ago I wrote about correlation IDs because we refused to accept "it's flaky" as a root cause for a request crossing five .NET services. We widened log windows, followed IDs, found the rolled-back transaction hiding in the noise. The same fight has come back, except this time the misbehaving component bills by the token and improvises its own control flow. The old excuse was that you simply could not see inside these systems. That excuse is expiring, and this post is about what replaced it.

<!--more-->

### One Question, Twelve Spans

A user question to our advisory chatbot used to be, observability-wise, boring. Now it looks roughly like this:

```text
POST /api/advice
└─ agent.run
   ├─ retrieve_documents        vector search, top 8 chunks
   ├─ rerank                    cross-encoder over candidates
   ├─ chat gpt-4o               2,140 tokens in / 310 out
   ├─ tool.get_margin_rule      internal API, 0.8 s
   └─ chat gpt-4o               3,905 tokens in / 220 out
```

That is still a distributed system. It just happens to be one where a service can decide mid-request to go read something else first. Retrieval can fetch the wrong documents, the first model call can invent arguments for the tool, and the second call happily synthesizes over whatever survived. When the final answer is wrong, "which step failed" is exactly the question we spent the last decade learning to answer for microservices. The answer has not changed either: open the span tree and look.

### The gen_ai Vocabulary

The pleasant surprise of the past year is that this stopped being bespoke glue. OpenTelemetry has been baking semantic conventions for generative AI, and they all live under a `gen_ai.*` namespace:

- `gen_ai.operation.name`, `gen_ai.request.model`, `gen_ai.response.model` — what operation ran and against which model
- `gen_ai.usage.input_tokens` and `gen_ai.usage.output_tokens` — the billable truth, recorded right on the span
- finish reasons — did the model actually answer, or stop because it wanted a tool executed, or hit the length ceiling?
- metrics like `gen_ai.client.token.usage` for the dashboard crowd

With these, a model call becomes an ordinary span named `chat gpt-4o`. An agent turn is a parent span with tool executions and further model calls nested underneath. Instrument once and your Python agent shows up in the same trace view as your Spring Boot services, which in my day job is not a small thing.

One honest caveat, because the observability vendors will not tell you: as of mid-2026 every one of these attributes is still marked Development, not Stable, in OpenTelemetry's maturity ladder. The conventions were moved into their own dedicated repository this June and have not even shipped a tagged release there yet. Names have already shifted once mid-flight — `gen_ai.system` became `gen_ai.provider.name`. My advice is to use them anyway. The OTLP transport and the whole trace data model underneath are rock solid; a renamed attribute is a search-and-replace if you hide the constants behind thin wrapper functions. The alternative is going back to every vendor inventing its own attribute soup, and I lived through that decade once already.

### Tokens Are Money

Here is what changes when token counts land on spans instead of in some vendor's billing tab: cost stops being weather and becomes a metric you can slice. Multiply usage by a price table and you get cost per feature, per user, per prompt template. In a bank the chargeback story writes itself, but the part I actually care about is anomaly detection. Every runaway agent I have seen announced itself the same way — input tokens climbing with every turn because a retry loop kept stuffing the growing transcript back into context. On a dashboard that is a hockey stick you can alert on before finance alerts you.

This is the same instinct from my cloud-bill FinOps days, moved up one abstraction level. The resource being wasted is smaller and weirder, but the discipline is identical: attribute it, graph it, embarrass the outliers.

### A Span Cannot Tell You If It Was Good

Now the uncomfortable part. All the machinery above tells you the call succeeded — HTTP 200, 220 output tokens, four seconds. None of it tells you the margin rule quoted to the user was correct. Latency and cost are solved problems; quality is the new kid on the telemetry block, and the only practical move I have seen is to treat quality as a signal that gets attached to the trace. Thumbs down from the user lands on the conversation record. A judge model grades a sampled slice of production traffic and the score sits next to the latency. Offline eval runs write their results onto the same object. Then, when somebody swears the bot gave a nonsense answer on Tuesday, the unit of investigation is a concrete trace with its scores and its span tree, not a recollection.

Even better: the bad traces become regression tests. Collect them, fix the pipeline or the prompt, re-run the set. That loop is doing more for our answer quality than any model upgrade this year.

### Prompts Are Logs You Did Not Mean to Keep

One design choice in the conventions deserves applause: capturing actual prompt and completion content is strictly opt-in. Every attribute capable of holding user text requires you to explicitly switch it on. In finance that default is exactly right, because prompts are magnets for customer identifiers and half-drafted risk commentary nobody wants sitting in a third-party trace store forever. Our compromise: structure and token counts always on, payload capture on in staging where the interesting failures live anyway, and heavy redaction if payloads ever need to ship from production. Ninety percent of debugging needs the shape of the request, not the transcript.

None of this is exotic anymore, is my overall point. Which step failed, what did it cost, was the answer any good — if your agent stack cannot answer those three questions, the gap is instrumentation work, not mystery. And the next time someone closes an incident with "the model hallucinated", you have my permission to reply the way a senior engineer would in 2020: show me the span.
