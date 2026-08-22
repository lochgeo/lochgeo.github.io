---
layout: post
title:  "FinOps - When the Cloud Bill Stops Being Weather"
date:   2022-10-16 14:22:00
comments: True
categories: [Software, Cloud]
excerpt_separator: "<!--more-->"
---

Finance forwarded the Azure invoice again this week. Not a polite FYI — a "can you explain the jump" thread with three managers CC'd. The honest answer was ugly: we could name the subscriptions, not the owners. Half the line items were tags like `rg-temp-demo` and a node pool nobody had touched since the last migration spike.

That used to be background noise. Cloud was the flexible thing that scaled when we shipped. This fall it feels different. Budgets are tighter, leadership is asking about ROI, and the bill is no longer weather that just happens. It is a product decision we forgot to staff.

<!--more-->

### What FinOps actually is (not another dashboard)

[FinOps](https://www.finops.org/) is the name the industry settled on for cloud financial management as a practice, not a once-a-quarter spreadsheet. The [FinOps Foundation](https://www.finops.org/about/) has been under the Linux Foundation since mid-2020. The useful bit is not the branding. It is the insistence that engineering, finance, and product sit in the same room with the same numbers.

Their loop is deliberately boring:

- **Inform** — allocate spend so a team can see *their* meters, not a company blob.
- **Optimize** — right-size, kill idle, buy commitments where the baseline is real.
- **Operate** — budgets, alerts, and a rhythm so cost is a weekly habit, not an autopsy.

I used to treat cost as ops theater. Then I watched a "temporary" AKS cluster outlive three feature releases and quietly beat the app's own compute line. Nobody was evil. Nobody owned the meter.

### Why this autumn feels different

Cloud spend has been climbing for years. What changed is the tone of the questions. Surveys this year keep saying the same thing in different words: a lot of companies moved fast, over-provisioned, and are now hunting for ROI they cannot point to on a slide. Flexera's 2022 State of the Cloud work put self-reported waste in the ballpark of a third of cloud spend. Take the exact percentage with salt — people underestimate waste — but the direction matches what I see in architecture reviews.

The metaphor that stuck for me: cloud is a taxi with the meter always running, not a company car you bought once. Leaving the meter on overnight used to feel like convenience. Now it feels like leaving the engine idling in the driveway while fuel prices climb.

### What actually moved our numbers

We did not stand up a FinOps "center of excellence" with a logo. We did three unglamorous things on the .NET estate I live in.

**1. Tags that survive a PR review.**  
`app`, `team`, `env`, `cost-center`. Missing tags fail the pipeline the same way a broken build does. Shared platform costs get an explicit bucket and a simple split rule so they do not vanish into "infrastructure." Without allocation, every optimization conversation is philosophy.

**2. A dashboard each squad will actually open.**  
Azure Cost Management (or AWS Cost Explorer if that is your house) filtered to *their* tags, not the enterprise rollup. Weekly budget alerts at 50 / 80 / 100 percent of a number product agreed when things were calm. Anomaly noise is real; we still prefer a slightly noisy alert over a surprise invoice.

**3. A kill list before a commitment shopping spree.**  
Idle public IPs, forgotten load balancers, non-prod that runs 24/7 for a team that works 10 hours a day, Premium SKUs on internal tools that get twelve requests a day, disk snapshots from migrations that finished in March. Then — and only then — Reserved Instances or Savings Plans on the steady baseline. Commitments on waste just lock the waste in at a discount.

On Kubernetes the same story shows up as requests and limits I already wrote about last year: if every pod asks for a full CPU "just in case," the cluster autoscaler buys air. Rightsizing is FinOps with a YAML file.

### Rate vs usage (who pulls which lever)

A clean split I stole from practitioners who do this full time:

- **Rate** (what you pay per unit) — enterprise agreements, Reserved Instances, Savings Plans, Azure Hybrid Benefit. Centralize this. One person thrashing commitment purchases across twenty teams is cheaper than twenty teams guessing.
- **Usage** (how many units you burn) — architecture, idle time, over-sized SKUs, chatty chatty chatty logging that turns App Insights into a second payroll. Decentralize this. Only the team that owns the ASP.NET Core service knows whether that always-on worker can become a queue + timer function.

If leadership only celebrates reserved-instance coverage, engineers will stop looking at design. If leadership only yells at engineers and never buys the baseline commit, finance will keep paying on-demand tax. You need both levers.

### Mistakes I have already made

- **Cost as a quarterly surprise.** By the time the invoice lands, the PR that caused it has merged, deployed, and been forgotten. Feedback has to be days, not months.
- **Optimization theater.** A big rightsizing week, a hero slide, then six months of silence while new services ship fat defaults again. FinOps without a backlog owner dies.
- **Unit-less graphs.** "We spent less" is weak if traffic doubled. Cost per order, per active user, or per thousand API calls is the conversation product understands. Pick one unit and stick with it long enough to trend.
- **Shame culture.** Publishing a leaderboard that humiliates a team for a migration spike teaches people to hide spend, not fix it. Pair the number with a named helper and a time-boxed fix.

### My take

FinOps is not a tool you buy and close the ticket. It is the boring discipline of making variable cloud cost as visible as latency and error rate — and giving the people who can change the architecture a reason to care before finance escalates.

If I had to start Monday on a mid-size .NET shop, I would not open with a multi-cloud FinOps platform RFP. I would pick the noisiest subscription, enforce five tags, ship one team-scoped cost view, put budgets on it, and schedule a thirty-minute "what did we burn and why" with product every two weeks. Commitments come after the idle is gone.

The cloud bill was never weather. We just stopped looking at the sky until it started raining money.
