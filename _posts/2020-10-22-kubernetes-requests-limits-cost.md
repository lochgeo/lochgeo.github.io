---
layout: post
title:  "Kubernetes Requests and Limits - Paying for Air"
date:   2020-10-22 19:32:00
categories: [Containers & Kubernetes]
excerpt_separator: "<!--more-->"
---

Our cloud bill went up again last month. Not because traffic exploded — traffic was fine. The cluster just... got fatter. More nodes. Same apps. Same user count. When I dug into it, the villain was not a runaway Deployment. It was a dozen YAML files where someone typed `cpu: "2"` and `memory: 4Gi` because that felt safe six months ago and nobody ever looked again.

Kubernetes will happily reserve capacity you never use. The scheduler only cares about *requests*. Your finance team only cares about the invoice. Those two facts meet in the middle as slack — resources booked, paid for, and idle.

<!--more-->

### What requests actually buy you

I used to treat `resources.requests` as a vague hint. It is not. It is the number the scheduler uses when it decides whether a pod fits on a node:

```
free ≈ node allocatable − sum(pod requests on that node)
```

Actual CPU usage on the box can be 15%. If the sum of requests says the node is full, the next pod stays `Pending` and Cluster Autoscaler (if you have it) buys another machine. You just paid for air.

Limits are the ceiling. CPU past the limit gets throttled. Memory past the limit can get the process OOM-killed. Different failure modes, same lesson: set them on purpose or the platform will set consequences for you.

### The mistakes I keep seeing (including mine)

A few patterns that show up every time I open a namespace that has not been touched in a while:

- **Copy-paste Guaranteed.** Someone set request = limit = a big round number "so it is QoS Guaranteed" and never measured. Stable, yes. Expensive, also yes.
- **No requests at all.** BestEffort pods look thrifty until the node is under pressure and they are the first to die. Fine for a throwaway job. Not fine for the API that takes payments.
- **CPU limits that throttle under normal load.** CFS quota can slow a container even when the node still has spare cores. I have chased "intermittent latency" that was just a tight CPU limit fighting a bursty .NET GC and thread pool.
- **Memory request much lower than real usage.** Works until a noisy neighbor or a traffic spike. Then eviction. Then a war room about "random restarts."
- **HPA on CPU with nonsense requests.** Horizontal Pod Autoscaler targets utilization *relative to requests*. If the request is fantasy, the scale-out math is fantasy too.

None of this needs a new product. It needs a habit of looking at what the process actually consumes.

### A practical starting point for a .NET API

I am not going to pretend one YAML fits every service. For a typical ASP.NET Core API we run behind a load balancer, the shape that has worked as a *starting* point — then we measure — looks like this:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 512Mi
  limits:
    cpu: "1"
    memory: 512Mi
```

Why that shape:

- **Memory request = memory limit.** Memory is not compressible. If the process needs 512Mi under load, reserve 512Mi. Overcommit on RAM is how you get surprise OOM kills when the node is tight.
- **CPU request modest, limit higher (or sometimes no CPU limit).** CPU *is* compressible. A lower request packs pods better; a higher limit lets short bursts ride unused cores. If you set a CPU limit, watch throttling metrics — `container_cpu_cfs_throttled_seconds_total` style signals — not just average usage.
- **HPA target in the 60–70% of request range** once metrics-server is healthy, with a floor of a few replicas so one node loss does not take the service with it.

Load-test one pod with autoscaling off first. Know how many RPS it holds before latency falls over. *Then* set requests. Guessing in YAML and "tuning in prod" is how the bill grows quietly for a quarter.

### QoS classes in plain language

Kubernetes derives a Quality of Service class from how you set requests and limits. You do not pick the label; you earn it:

- **Guaranteed** — every container has CPU and memory limits, and requests equal limits. Last to be evicted when the node is desperate.
- **Burstable** — at least one request set, but not the full Guaranteed pattern. Can use spare capacity; more exposed under pressure.
- **BestEffort** — no requests or limits. Cheap until the node hurts.

For anything user-facing I want Guaranteed or a carefully sized Burstable with real memory requests. BestEffort is for batch you can replay, not for the thing on-call gets paged about.

### Cost levers that actually move the needle

Right-sizing pods is step one. A few others that were already common sense by mid-2020:

- **Cluster Autoscaler + honest requests.** Autoscaler scales nodes from pending pods and low utilization. Garbage requests → garbage node count.
- **Do not run Friday's traffic shape on Sunday.** Dev and internal tools do not need full replica counts at 3am. Scheduled scale-down (or a simple off-hours replica patch) is boring and effective.
- **Spot / preemptible for the right workloads.** Great for stateless workers that retry. Bad for the database you have not failed over in anger yet.
- **Namespace ResourceQuotas.** When multiple teams share a cluster, quotas stop one enthusiastic chart from eating the node pool. LimitRanges that inject defaults beat a wiki page nobody reads.

AWS and GCP both published cost-optimization writeups around this space last year and this year — Cluster Autoscaler, HPA, right-sizing, purchase options. The tools differ; the physics do not. Reserved capacity you do not use still shows up on the invoice.

### How I review a noisy namespace

When a namespace "feels expensive," I do not start with a new platform tool. I start with evidence:

1. List Deployments missing requests entirely — fix those first.
2. Compare request vs actual usage over a week (Prometheus, Metrics Server, cloud console — whatever you already have). Big gap = slack.
3. Check CPU throttling on services with tight limits and latency complaints.
4. Check HPA: is it flapping because the request is tiny and one extra concurrent request looks like 200% utilization?
5. Only then talk about bigger nodes, more nodes, or a different instance family.

The embarrassing win is almost always the same: lower a few fantasy requests, raise a couple of starving ones, and watch pending pods and node count both calm down.

### What I want us to stop doing

Stop treating resource fields as ceremony you fill so the PR checklist goes green. They are capacity planning expressed in YAML. Wrong numbers do not fail the deploy. They fail slowly — as a thicker node group, a slower scale-up, or a 2am eviction that looks like an application bug.

If I had to pick one rule for our .NET services on Kubernetes right now: **measure one pod under realistic load, set memory request equal to what it needs with headroom, set CPU request from steady-state and let limits (or no CPU limit) handle bursts, then revisit after a week in production.** Not after the next invoice shock.

Paying for capacity you use is engineering. Paying for air is just a YAML habit we can break.