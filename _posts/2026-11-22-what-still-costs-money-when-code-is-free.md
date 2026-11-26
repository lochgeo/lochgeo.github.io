---
layout: post
title:  "What Still Costs Money When Code Is Free"
date:   2026-11-22 05:20:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

Somebody in a planning call last month said we should start assuming software is basically free. I didn't argue, because the premise is defensible. A decent model writes the boilerplate, the glue, the tests and half the front end for a few dollars of tokens. Two years ago that same work was a sprint.

But software was never only the writing. It still has to solve a real problem, survive contact with users, fail in ways someone can understand, and be changed six months later by a person who was not in the room. The cheaper code becomes, the more those surrounding jobs matter.

<!--more-->

### The part that did get cheap

Let me give the cheap side its due, because it's real.

OpenAI's GPT-6.2 Astra, launched on 17 November, costs $8 per million input tokens and $40 per million output tokens, with cached input at $0.40 — 20% below GPT-6 Astra on input and output, and a 60% discount on cache. Google's Gemini 4 Neon, out a day later, sits at $3 and $15 with a million-token context and cached input at $0.15.

Those are real reductions. But a price per token is not the cost of trusted software. A plausible diff that needs two rounds of rework, an hour of review and a migration nobody understands is not cheap just because the model call was. The unit cost of generation is falling faster than the total cost of getting a change safely into the world.

### The physical floor is still real

One physical bill sits underneath the token price. Nikkei reported in early November that DRAM prices had roughly doubled from the end of last year as manufacturers prioritised AI datacenter supply. High-bandwidth memory makes the tradeoff unusually direct: every bit of HBM produced is ordinary memory that is not. The International Energy Agency, meanwhile, expects datacenter electricity use to roughly double by 2030, even as energy per task keeps falling. Cheaper inference expands the menu of things we ask machines to do, and new grid connections are measured in years of permitting. You cannot generate a transmission line.

Both constraints matter for capacity planning and architecture. But they are one supporting point, not the main story. The more immediate constraint for most engineers is attention.

### Code is abundant. Trust is not.

The bottleneck I feel most in my own week is review.

Faros AI's 2026 engineering report compared teams with low AI adoption to teams with high adoption. Median time to first PR review more than doubled, time in review more than quadrupled, and lead time from commit to production rose nearly fivefold. Those comparisons do not prove AI caused every minute of that delay, but the direction is hard to ignore.

LeadDev's survey of 377 engineers and leaders found 45% saying review capacity is the biggest drag on delivery, and only 26% claiming they fully understand the AI code they review. Sonar's numbers point the same way: 61% agree the code often looks correct but isn't reliable.

That is the shift. Producing a plausible change has become easy. Knowing whether it belongs in the system is still expensive. The reviewer has to reconstruct the intent, check the assumptions, notice the missing edge case, and understand how one more “small” abstraction will behave under load six months from now.

When output is abundant, plausibility is abundant too. Trust does not scale with it.

### What senior engineers should focus on

I don't think the answer is simply to become better at prompting. Seniority was always less about how quickly someone could produce code than about how well they could decide what deserved to exist and carry the consequences. AI makes that distinction impossible to ignore.

**Own the outcome, not the diff.** An agent can implement a feature. Someone still has to decide whether it is the right feature, which constraints matter, what the migration path looks like, and what happens when it fails in production. Ownership is the work around the code: shaping the problem, making the tradeoffs explicit, and staying until the system behaves as promised.

**Design for verifiability.** “The tests pass” is no longer a strong enough guarantee. The work is to make correctness observable through invariants, contracts, useful tests, evals and telemetry. The question is not whether a human can memorise every generated line. It is whether the system gives us enough evidence to know why we trust it.

**Keep the debugging instinct alive.** Use the machine for the first draft and the boring search. Still reproduce the bug, follow the trace and understand the dependency that broke. If I only review plausible output, I slowly lose the mental model that lets me make a good decision when the happy path stops working. The model can hold the code; the senior engineer still has to hold the system.

**Make the people around you more capable.** A team does not get faster by producing more diffs than it can understand. Review quality, clear decisions, useful documentation and mentoring now matter as much as individual output. Juniors also need deliberate practice: real debugging, small ownership, and the chance to explain their reasoning rather than just accept whatever the agent returned. Otherwise we produce engineers who can ask for code but cannot diagnose it.

### The honest version

I don't think software is becoming free. I think creation is becoming cheap while decision-making, verification, operation and teaching become more valuable.

The model prices will keep falling. Memory, power and the reviewer's afternoon will still have to be paid for. But the scarce resource that will shape my next two years is not a wafer or a megawatt. It is the attention of people who can tell the difference between code that looks finished and software that is actually ready.
