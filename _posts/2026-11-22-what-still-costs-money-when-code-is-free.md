---
layout: post
title:  "What Still Costs Money When Code Is Free"
date:   2026-11-22 05:20:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

Somebody in a planning call last month said we should start assuming software is basically free. I didn't argue, because the premise is defensible. A decent model writes the boilerplate, the glue, the tests and half the front end for a few dollars of tokens. Two years ago that same work was a sprint.

But "free" is the wrong model. Software still has to sit in memory, run on electricity, and get signed off by a person who is willing to put their name on it. I went looking at the numbers behind those three things this week, and not one of them got cheaper this year. Two got dramatically more expensive.

<!--more-->

### The part that did get cheap

Let me give the cheap side its due, because it's real.

OpenAI's GPT-6.2 Astra, launched on 17 November, costs $8 per million input tokens and $40 per million output tokens, with cached input at $0.40 — 20% below GPT-6 Astra on input and output, and a 60% discount on cache. Google's Gemini 4 Neon, out a day later, sits at $3 and $15 with a million-token context and cached input at $0.15.

Those are official list prices, and they're per token, not per task, so the effective saving is larger than the sticker suggests: both vendors are selling fewer output tokens for the same job. My own rough rule is that a task I would have handed to a mid-level engineer in 2024 now costs me less than the coffee I drink while reviewing the result.

That's the good news. Here's the bill.

### Memory is the first thing that got expensive

When demand for AI capacity goes up, the memory fabs don't build a second factory for your laptop. They reallocate wafers.

Nikkei reported in early November that DRAM prices had roughly doubled from the end of last year, because manufacturers are prioritising AI datacenter supply over commodity products like PCs. TrendForce's numbers behind that: average DRAM prices up 50–55% in the first quarter of 2026 versus the fourth quarter of 2025, a jump its own analyst called unprecedented.

The mechanism is easy to miss. High-bandwidth memory, the stuff stacked around an AI accelerator, has what Micron's business chief described as a three-to-one basis — every bit of HBM you make is three bits of ordinary memory you don't. TrendForce expects HBM to take roughly 30% of DRAM wafer input at the big three makers next year, up from 23% this year, and HBM4 prices are forecast at $3 to $5.50 per gigabit against $2.05 now.

So the price of memory in an ordinary laptop went from 10–18% of hardware cost in the first half of 2025 to about 20% now. Micron quietly exited the consumer PC memory business. Dell's COO said he didn't see how the shortage wouldn't reach customers.

Every cheap token I generate is, somewhere upstream, competing with a stick of RAM for a teenager's gaming PC. That's not a metaphor. It's the same wafer.

### Power is the second bill

The International Energy Agency's April report is the most sober thing I've read on this. Datacenter electricity use grew 17% in 2025, and AI-focused datacenters grew 50%. The base case takes datacenters from 485 TWh in 2025 to around 950 TWh by 2030, roughly 3% of global electricity.

Here's the part that should make any engineer pause: energy per AI task has been falling by at least an order of magnitude per year. A simple text query now uses less power than a television running for the same time. Efficiency is winning on every individual task, and total demand still doubles, because we keep finding heavier things to do — reasoning, agents, video.

That is Jevons' paradox with a meter attached. Cheaper inference doesn't shrink the bill, it expands the menu.

And the menu has lead times. The IEA notes a scramble for grid connections, and that the current HBM shortage is expected to persist through at least the end of 2027. CNBC's read of the IEA's latest scenario puts datacenter demand near 1,000 TWh by 2030, with 26 million kilometres of new transmission lines needed by 2035. You cannot generate a transmission line.

### The third bill is us

The one I feel most in my own week is review.

Faros AI's 2026 engineering report compared teams with low AI adoption to teams with high adoption. Median time to first PR review went up 156.6%. Average time in review went up 199.6%, and the median went up 441.5%. Lead time from commit to production rose 480%.

LeadDev's survey of 377 engineers and leaders found 45% saying review capacity is the biggest drag on delivery, and only 26% claiming they fully understand the AI code they review. Sonar's numbers point the same way: 38% say reviewing AI-generated code takes more effort than reviewing a colleague's, and 61% agree the code often looks correct but isn't reliable.

So the pipeline got faster and the queue got longer, because the last mile is a human deciding whether plausible code is actually correct. That last mile has no price cut coming. If anything it gets more expensive, because the diffs are bigger and the author often can't explain them.

### What stays scarce

Put those three together and the shape of the next few years is fairly clear. When production gets cheap, value moves to whatever production needs and cannot make:

- **Memory and silicon capacity** — allocated by contract, not by wish.
- **Power and grid connections** — measured in years of permitting.
- **Verification and judgment** — a senior engineer's attention, per diff.
- **Accountability** — someone whose name is on the change when the auditor asks.

The BLS still projects software developer employment up 15.8% between 2024 and 2034, about 268,000 more jobs. I don't read that as reassurance that nothing changes. I read it as the work moving toward the scarce side of the stack.

### What I'd try

1. Price your work by the scarce part, not the typing. Nobody pays for characters; they pay for someone who can say "this is correct and here's why."
2. Get good at verification specifically. Reading diffs, writing the test that would have caught it, knowing which invariants matter. That's the bottleneck I keep running into.
3. Put a number on your own cost stack — tokens, memory, review minutes, cloud GPU rate — even roughly. You can't reason about an economy you've never measured.
4. Watch the complements instead of the leaderboard. DRAM spot prices and your review queue length tell you more about next year than another benchmark table.
5. Own something end to end. Ownership is the one input an agent can't hold, and it's the reason the review queue exists.

### The honest version

I don't think software is becoming free. I think it's becoming cheap at the point of creation and expensive at every point around it, and most of the expensive parts are physical or human, with lead times measured in years rather than release cycles.

The model prices will keep falling. The memory, the megawatts and the reviewer's afternoon won't. If I'm planning my own next two years, I'd rather be the person who understands the expensive end.
