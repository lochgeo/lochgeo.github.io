---
layout: post
title:  "Backpressure - Learning to Say No Before the Queue Says It for You"
date:   2026-08-06 10:15:00 +0530
comments: True
categories: [Software, Architecture]
excerpt_separator: "<!--more-->"
---

It was a regular Tuesday morning when our settlement-status service fell over, and nothing about the traffic looked unusual at first. Volumes were maybe thirty percent above normal, the kind of spike we had survived before. But an upstream job had started retrying aggressively on timeouts, so every slow response came back as two more requests, and within twenty minutes the Tomcat thread pool was fully occupied with calls waiting on a downstream that was itself waiting. Health checks timed out. Kubernetes restarted pods that were not broken, just busy. The restart dropped the in-flight work, the retries noticed, and the whole thing repeated.

I spent that afternoon staring at a dashboard showing a queue depth climbing like a staircase and thinking about how polite our service had been through all of it. It accepted every request. It queued every request. And then it served none of them well. A bouncer who lets everyone into a packed room is not being kind. He is starting a stampede.

<!--more-->

### Queues are shock absorbers, not storage

The Azure Architecture Center has a pattern called queue-based load leveling that I keep coming back to: put a queue between the work arriving and the service doing the work, and let the service drain it at its own pace. Bursts get smoothed. Callers do not block waiting for a busy service. It is the same reason we put Kafka between our intake API and the settlement writer. On a normal day the consumer lag graph is flat and boring, exactly how I like it.

But every queue has a lie built into it, which is that buffering feels like handling. Little's law says it plainly: in steady state, the number of requests in the system equals arrival rate times service time. If arrivals outrun service for long enough, the queue does not stabilize, it grows, and each new arrival waits longer than the last until everything times out at once. The Azure guidance says the same thing with operational bluntness: monitor queue depth, scale consumers within safe limits, or shed work at the producer.

That Tuesday our queue did exactly what unbounded queues do. It held the work faithfully while the latency behind it went from seconds to timeouts, and by the time anything was shed, the callers had already given up. A queue without a bound and a policy is just a slower, more memory-hungry way to fail.

### Say no early, and say it politely

Load shedding sounds harsh, but it is the kindest thing an overloaded service can do. A fast 503 with a `Retry-After` header tells the caller to back off and try elsewhere. A slow timeout after thirty seconds tells the caller nothing and costs both sides everything.

Netflix learned this years before I needed it. Their 2020 writeup on prioritized load shedding in Zuul describes shedding by priority when error rates or concurrency cross thresholds, and their 2024 follow-up on the PlayAPI goes further: a partitioned concurrency limiter that favors user-initiated requests over prefetch background traffic, decided in a cheap servlet filter from a request header so rejecting costs almost no CPU. When the server is at its limit, the lowest-priority work goes first and the traffic that matters keeps flowing.

My version is much smaller, but the shape is the same. I decide the priority order before the spike, not during it:

- Interactive status checks and payment submissions get served first.
- Statement exports and report generation get queued or deferred.
- Bulk reprocessing and prefetch-style background sync get shed first, with a clear 503 and a retry hint.

In our Spring Boot services the enforcement is a Resilience4j bulkhead per class of work, with a bounded wait. When the bulkhead is full, the call fails fast with a `BulkheadFullException` that I translate to a 503, instead of joining a queue it will never leave:

```
resilience4j.bulkhead:
  instances:
    statusChecks:
      maxConcurrentCalls: 60
      maxWaitDuration: 50ms
    reports:
      maxConcurrentCalls: 10
      maxWaitDuration: 0ms
```

Zero wait on the low-priority bulkhead is deliberate. If there is no room right now, I would rather tell the report caller immediately than let it sit. The caller can retry in a minute. It cannot un-wait thirty seconds for a timeout that teaches it nothing.

### Bulkheads keep one slow caller from sinking the ship

The bulkhead pattern is named after the partitions in a ship's hull: one compartment floods, the ship stays afloat. The idea is to split a shared resource into isolated pools so one hungry consumer cannot eat everything.

Our concrete hungry consumer was the Tomcat thread pool. The default `server.tomcat.threads.max` is 200, and for years I treated that as headroom. That Tuesday taught me it is a single shared pool: every request thread parked on the slow downstream was a thread that could not serve a fast, easy status check. Two hundred threads sounds like a lot until one dependency turns them all into a waiting room.

So now the rule is simple: anything that calls a different backend gets its own bulkhead with its own limit, sized to what that backend can actually handle. The status-check path and the report path do not share concurrency budget anymore. A slow report query can saturate its own small pool and shed, while status checks sail past in theirs. Resilience4j and Polly both exist precisely for this kind of consumer-side isolation, and combined with timeouts they turn one big shared failure into several small contained ones.

### What I actually do now

None of this required new infrastructure. It required writing down the shedding order and enforcing it. The checklist taped next to my monitor:

- Every queue gets a bound and an alert. Kafka consumer lag has a threshold with a page attached, not a graph I admire the next morning.
- Admission control lives at the edge. Shed in the controller or filter, before the request holds a thread, a connection, and my optimism.
- The priority order is written down and reviewed. Shedding is a product decision about what matters, not a panic decision at 2 AM.
- Shed responses are a contract: 503 or 429, a `Retry-After`, and a metric I can see. Every shed request is counted, because silent shedding is just dropping work and hoping.
- We rehearse it. A load test that pushes past the limit on purpose, once a quarter, so the first time the limiter fires is not the first time we trust it.

Circuit breakers got their own post years ago on this blog, and they are still the right tool for a failing dependency. But breakers trip after failures. Backpressure and shedding decide beforehand how much trouble you will accept. One stops the bleeding, the other keeps you out of the fight you cannot win.

My opinion, earned on that Tuesday: a service that says no quickly and clearly is more reliable than one that says yes to everything and means none of it. Bound the queues, partition the pools, write down who gets shed first, and let the spike pass through instead of through you.
