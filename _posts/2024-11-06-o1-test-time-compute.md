---
layout: post
title:  "o1 - When the Model Starts Thinking Before Answering"
date:   2024-11-06 09:47:00 +0530
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

OpenAI dropped o1-preview and o1-mini in mid-September. Two months in, the hype has settled into something more useful: a clearer picture of where these "reasoning models" actually fit in a practitioner's toolkit — and where they don't.

I've been building LLM-backed features inside a regulated financial institution for the better part of this year. That means every model choice runs through a mental checklist: latency budgets, cost per thousand calls, data residency, audit trails, and whether the thing hallucinates less than the last one. o1 is the first model that made me update that checklist in a while.

<!--more-->

### What changed

The short version: o1 doesn't just predict the next token. It generates a hidden chain of thought — "reasoning tokens" — before emitting the visible answer. You pay for those tokens (they count as output), you don't see them, and they're why a single query can take 30 seconds and cost 4x what GPT-4o charges.

OpenAI's benchmarks show o1 crushing GPT-4o on math, coding, and science exams. Codeforces 89th percentile. AIME qualifier territory. GPQA diamond above PhD level. Impressive numbers. But benchmarks aren't production workloads.

### The trade-offs you feel in prod

**Latency is the big one.** A straightforward "explain this regex" takes GPT-4o maybe two seconds. o1-preview can sit there for twenty, thirty seconds "thinking." For a batch job that runs overnight? Fine. For a chat feature where a compliance analyst is waiting? Painful. We've already had product folks ask why the "smart model" feels slower than the "dumb one."

**Cost scales differently.** o1-preview runs $15/M input, $60/M output. GPT-4o is $5/$15. The reasoning tokens are the killer — a complex coding task can burn 20k+ output tokens where 4o would use 3k. At our volume, that's a real budget conversation. o1-mini at 80% cheaper ($3/$12) helps, but it's not a drop-in replacement; it lacks broad world knowledge.

**No tool use, no streaming, no system prompt.** The API surface is deliberately stripped down. You get user/assistant messages only. If your agent loop needs function calling — and ours does, for pulling policy documents and running calculations — you're stuck orchestrating that outside the model. That means more glue code, more failure modes.

**The "thinking" is opaque by design.** OpenAI shows a summarized reasoning trace in ChatGPT. In the API, you get nothing. You can't inspect, log, or audit the chain of thought. In a regulated environment where "why did the model say that?" is a compliance question, that's a gap. We've had to build fallback logging that captures the prompt, the answer, and a disclaimer: reasoning trace unavailable.

### Where it actually earns its keep

Despite the friction, there are genuine wins:

- **Complex multi-step refactors.** "Here's a 400-line service, split it into three modules with clear interfaces, add unit tests, keep the same behavior." o1-preview gets closer to a working result in one shot than 4o with three rounds of prompting.
- **Novel algorithm implementation.** "Implement the Hungarian algorithm for this assignment problem with these constraints." It produces correct, commented code more often than not.
- **Debugging nasty logic bugs.** Paste a failing test and the suspect function; o1 often spots the off-by-one or the edge case that three junior devs (me included) missed.

The pattern: tasks where the *reasoning depth* matters more than *context breadth*. Where you'd happily wait thirty seconds for a correct answer rather than get a plausible wrong one in two.

### What I'm still figuring out

**Routing logic.** When does a request go to o1 vs 4o vs 4o-mini? We're experimenting with a classifier: if the prompt contains "debug," "refactor," "prove," "optimize," or exceeds a token threshold, route to o1-mini first. Escalate to o1-preview only if mini fails. Early days, but it keeps costs from spiraling.

**Evaluation.** How do you eval a model whose behavior changes with "thinking time"? OpenAI's plots show performance improving with test-time compute, but they don't expose a knob for it yet. For now we're treating o1 as a distinct model tier with its own eval set — separate from the 4o regression suite.

**Local alternatives.** The open weights community is chasing this. Projects like r1-light and various CoT-distilled Llamas are trying to replicate the reasoning pattern without the API tether. Nothing production-ready yet, but the trajectory is clear: test-time compute scaling is the new frontier, and it won't stay closed-source forever.

### The verdict (for now)

o1 isn't a drop-in upgrade. It's a specialized tool for a specific class of problems — the ones where you'd pay a senior engineer to stare at a whiteboard for twenty minutes. If your workload is summarization, classification, extraction, or straightforward Q&A, stick with 4o or 4o-mini. You'll save money, latency, and architectural headache.

But if you're building something that needs genuine multi-step reasoning — code generation with complex constraints, mathematical derivation, logic-heavy analysis — o1-preview is the first model that feels like it's *working the problem* rather than *pattern-matching the answer*. That distinction matters.

I'll keep routing the hard stuff to o1-mini, keep watching the local model space, and keep asking every new model: "what's the latency budget, and who pays for the reasoning tokens?" Some questions don't change.

{% include pullquote.html quote="Test-time compute is the new scaling law — and it comes with an invoice." %}