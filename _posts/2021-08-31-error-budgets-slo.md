---
layout: post
title:  "Error Budgets - Permission to Ship, Permission to Stop"
date:   2021-08-31 19:36:00
comments: True
categories: [Observability]
excerpt_separator: "<!--more-->"
---

We spent the first half of the year getting better at *how* code lands in prod — pipelines, GitOps, flags so deploy and release are not the same button. The question that keeps coming up in incident reviews is different: when are we *allowed* to keep shipping, and when should we put the brakes on?

Uptime dashboards full of green do not answer that. Neither does "we should be more careful." What finally helped our team was treating reliability like a budget you can spend, not a vibe you argue about after every outage.

<!--more-->

### SLI, SLO, error budget — without the jargon fog

I had read bits of Google's SRE material for years and still mixed the terms up in meetings. The clean split that stuck for me:

- **SLI** — the thing you measure. Ratio of successful requests, latency under a threshold, freshness of a batch job. A number from real traffic, not a hope.
- **SLO** — the target for that SLI over a window. "99.9% of checkout API calls succeed over 28 days." Internal. Ours to defend.
- **SLA** — the contract with teeth (credits, penalties). Usually looser than the SLO so you have room before legal pain.
- **Error budget** — whatever is left when you subtract the SLO from 100%. A 99.9% availability SLO is a 0.1% budget for bad events.

If you take a million user-facing requests in the window, 0.1% is a thousand failures you can "afford" before you have broken your own promise. That number is not permission to be sloppy. It is a shared currency for the trade-off everyone already makes in their head: ship features vs harden the path.

Google's [SRE book](https://sre.google/sre-book/table-of-contents/) and the later [SRE Workbook](https://sre.google/workbook/table-of-contents/) are still the clearest write-ups I know. The workbook's [example error budget policy](https://sre.google/workbook/error-budget-policy/) is basically a one-pager that says: above SLO, ship as usual; budget blown for the rolling window, halt risky change until you are back.

### What we tried on a .NET API

We did not stand up a full SRE org. We picked one user-facing ASP.NET Core service — the one that pages people on a Friday — and wrote three lines everyone could repeat:

1. **SLI:** HTTP responses that are not 5xx and complete within 2 seconds, counted at the edge (ingress / reverse proxy), not inside the pod where a crash can hide.
2. **SLO:** 99.5% over a rolling 28 days. Not 99.99%. We were not Google, and lying with nines only trains leadership to ignore the number.
3. **Policy:** if the budget is exhausted, no non-essential deploys until we recover or an explicit exception is signed. Security patches and P0 fixes still go.

Measurement was boring on purpose. We already had request metrics. We added a simple good/bad counter and a small dashboard: remaining budget, burn over the last hour, burn over the last day. Correlation IDs (I wrote about those last year) made it easier to sample the failures that actually ate budget instead of arguing from anecdotes.

The first useful fight was over *what counts as bad*. Synthetic health checks that hit a shallow `/health` every second were inflating "success" while real users hit a slow dependency path. We moved the SLI to the authenticated business route. Suddenly the budget matched the tickets.

### The policy is the point

A number without consequences is a wallpaper SLO. The hard part was agreeing, in writing, what happens when the bar turns red:

- **Budget healthy** — normal release train. Feature flags still dark until product says go.
- **Burning fast** — page on-call, incident mode, freeze *feature* deploys for that service until the bleed stops. You can still ship a fix.
- **Budget empty for the window** — reliability work jumps the backlog. Postmortems are not optional if one incident ate a big chunk of the budget (the workbook suggests thresholds like ~20% of the window from a single event — we used that as a starting point, not gospel).

Product signed it. That mattered more than the math. Before, every freeze felt like ops being difficult. After, the freeze was the policy *they* had approved when things were calm.

I think of it like a team lunch budget. You can order the fancy place early in the month. If you blow it on Monday, you are on canteen food until the next cycle — and nobody pretends the spreadsheet is being dramatic.

### Mistakes we made in the first month

- **Too many SLOs.** We drafted eight. Nobody looked at seven of them. One or two user journeys beat a spreadsheet of vanity nines.
- **Alerting on raw error rate only.** A brief spike and a slow week-long leak need different responses. We are moving toward burn-rate style thinking (how fast the budget drains), not just "errors > 1% right now."
- **Blaming only "our" code.** A shared platform outage burned budget we could not fix in the app repo. The policy now says: if the cause is clearly upstream and that team has frozen too, we do not self-flagellate — but we *do* record the dependency and ask whether the SLO should assume less from them.
- **100% as the quiet goal.** If the real target is "never fail," you will never spend budget on change, and you will still fail. An honest SLO below perfection is kinder to on-call and more honest with users.

### My take

Error budgets did not make our systems more reliable by magic. They made the argument shorter. Ship or stabilize stopped being a personality contest and became a number both sides could see.

If you only do one thing: pick a single customer-facing endpoint, define one SLI you can measure at the edge, set a slightly uncomfortable SLO, and write a three-bullet policy for what happens when the budget is gone. Put it where product and eng both look. Then keep the dashboard honest for a full window before you tighten anything.

Reliability is not the absence of failure. It is knowing how much failure you agreed to, and changing your behaviour when you have spent it.
