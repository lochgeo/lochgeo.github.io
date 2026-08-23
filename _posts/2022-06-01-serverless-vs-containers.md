---
layout: post
title:  "Serverless vs Containers - Stop Picking Sides"
date:   2022-06-01 11:14:00
comments: True
categories: [Cloud]
excerpt_separator: "<!--more-->"
---

Every architecture review this year seems to end the same way. Someone draws a box labeled "Lambda" or "Azure Functions," someone else draws a Kubernetes cluster, and the room splits into camps. One side talks about never patching a server again. The other side talks about cold starts and the 15-minute wall. Both are right about *something*. Neither is right about everything.

I write ASP.NET Core services for a living. I have shipped the same kind of work as a zip-deployed function, as a container on a managed orchestrator, and as a plain old App Service. The choice stopped being ideological for me the day a "cheap" serverless path cost more than the always-on pod it replaced — and the day a "simple" cluster burned a week of ops time for a webhook that runs twice an hour.

<!--more-->

### What people actually mean by serverless

In practice, when engineers say serverless in 2022 they usually mean **Functions as a Service**: AWS Lambda, Azure Functions, Google Cloud Functions. You bring the handler. The platform brings the scale-to-zero, the patching, and the bill that is mostly invocation plus GB-seconds.

There is a second, fuzzier meaning — "I run containers but I do not manage nodes" — Fargate, Cloud Run, Azure Container Instances. That is still ops-light. It is not the same product as a 200-line function with an event trigger. Mixing the two words in a design doc is how you get a three-hour meeting about nothing.

I keep the labels boring on purpose:

- **FaaS** — short, event-driven units. Scale to zero. Hard platform limits.
- **Containers on a platform** — your process, your image, someone else's nodes (or none you can SSH to).
- **Containers you operate** — Kubernetes, ECS on EC2, the full DIY.

### The constraints that actually decide

Marketing slides talk about agility. Production talks about ceilings.

**AWS Lambda** (the one I keep running into on multi-cloud diagrams) is blunt about the box you live in:

- Max execution time is **15 minutes**. Past that you need Step Functions, a queue worker elsewhere, or a real service.
- Package as a zip (with layers) or, since late 2020, as a **container image from ECR** up to a much larger size — useful when NuGet and native deps stop fitting the old zip story.
- Cold starts are real, especially for .NET. **Provisioned concurrency** has existed since late 2019 if latency SLOs demand it; you pay to keep environments warm whether traffic shows up or not.
- API Gateway still has its own timeout story. A long Lambda timeout does not magically make a synchronous HTTP client wait forever.

**Azure Functions** is where a lot of my .NET day job gravity sits. Runtime 4.0 with **.NET 6** went GA in late 2021 for both the classic **in-process** model and the **isolated worker** model. Isolated is the one that feels like a normal host — your own `Program.cs`, your own DI, fewer "why is this package fighting the host" nights. Consumption plan is the true scale-to-zero path; Premium and dedicated plans trade money for always-on instances and VNet comfort.

None of that is a verdict. It is a checklist. If your workload fails the checklist, stop forcing FaaS.

### When functions win

I reach for functions when the shape of the work matches the platform:

- **Spiky or idle most of the day.** Nightly file drops, webhook receivers, queue consumers that sleep between bursts. Paying for idle vCPU feels silly.
- **Clear event boundaries.** S3/Blob created, Service Bus message, Event Grid, timer. One trigger, one job, write the result somewhere durable.
- **Small surface area.** A few dependencies, fast startup, no long-lived sockets you secretly rely on.
- **Team bandwidth is the scarce resource.** Nobody wants another Deployment, HPA, and Ingress just to resize a thumbnail.

A concrete pattern that keeps working for me: API or UI drops work on a queue; a function does the slow part; the caller polls or gets a callback. You stay inside timeouts. You do not pretend a 12-minute report is a synchronous GET.

### When containers win

I reach for containers when the work looks like a *service*, not a *reaction*:

- **Sustained traffic.** If the function would be warm all day anyway, the "scale to zero" discount is already gone. Compare real monthly numbers, not the free-tier slide.
- **Longer than the FaaS ceiling**, or chatty multi-step work that hates being sliced into orchestrator glue.
- **Custom runtime / native deps / sidecars** that fight the function packaging model even with container images.
- **You already run Kubernetes well.** Adding one more Deployment is cheaper than inventing a second ops culture for a handful of functions.
- **Stateful adjacent pieces** — anything that wants sticky sessions, big local cache, or a daemon next to the app. FaaS will make you externalize all of that; sometimes that is good design, sometimes it is just tax.

Fargate and friends sit in the middle: container packaging without node babysitting. Great for teams that already think in Dockerfiles but do not want etcd drama. Still not free at idle if you keep tasks running.

### The cost story nobody puts on the first slide

Serverless is cheap until it is not. Containers are expensive until they are not.

Rough mental model I use in reviews:

1. **Estimate steady-state RPS and average duration.** Multiply. That is your FaaS compute shape.
2. **Add the always-on floor** you will actually buy — provisioned concurrency, Premium plan minimums, NAT gateways for VPC functions, App Insights, log ingest. The function line item is rarely the whole bill.
3. **Compare to the smallest boring container** that meets the latency SLO (one or two tasks/pods, right-sized CPU/memory).
4. **Price the people.** A clever serverless design that three engineers understand is more expensive than a dull Deployment the whole team can page on.

I have seen low-traffic internal tools thrash money on a mini AKS cluster. I have also seen a chatty .NET API on Lambda with provisioned concurrency cost more than the two Fargate tasks it replaced, once cold-start pain forced the concurrency dial up. Neither outcome was in the original deck.

### How I decide in a design review

I ask five questions and try to answer them out loud:

1. What is the **P99 duration** and is it under the platform timeout with room to spare?
2. Is traffic **spiky with real idle**, or "always a little busy"?
3. Can we live with **cold starts**, or do we pay to erase them?
4. Does the team already have a **container platform** with decent CI, observability, and on-call habits?
5. Are we choosing this because it fits the workload — or because last conference keynote said so?

If (1)–(3) scream FaaS and (4) is "we barely operate what we have," functions. If (1)–(3) scream service and (4) is "we already run k8s," containers. If the room cannot answer (5) without marketing words, postpone the decision and prototype both with a real traffic shape for a week.

### My take

Serverless vs containers is a false war. Functions are outstanding for event-shaped work with honest limits. Containers are outstanding for long-running services and teams that already paid the orchestration tax. The boring hybrid — APIs and workers on containers, glue and spike handlers on functions — is what most grown-up systems look like when nobody is performing for a slide.

I still like scale-to-zero. I still like a Dockerfile I can run on my laptop. I refuse to pretend one of those preferences is an architecture. Match the runtime to the traffic, the timeout, and the team you actually have — not the one in the conference talk.
