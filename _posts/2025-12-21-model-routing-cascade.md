---
layout: post
title:  "Model Routing - Stop Sending Everything to the Expensive Model"
date:   2025-12-21 09:47:00 +0530
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

Not long ago, I watched a teammate's agent loop burn through $2,300 in API calls overnight. The task? Classifying support tickets into three buckets. The model? GPT-4o. Every single ticket — even the "reset my password" ones — got the full frontier treatment.

That's the trap. When you first get API access to a reasoning model, it feels like a superpower. You start reaching for it everywhere. Why wouldn't you? It's smarter, it reasons, it gets the hard stuff right. But somewhere between the demo and the production bill, the economics invert. You're paying premium prices for commodity work.

<!--more-->

### The pricing cliff is real

As of late 2025, the gap between model tiers is staggering. You're looking at roughly:

| Tier | Examples | Ballpark $/M tokens (blended) |
|------|----------|-------------------------------|
| Premium reasoning | GPT-4o, Claude 3.5 Opus | $30–60 |
| Mid-tier capable | GPT-4o-mini, Claude 3.5 Sonnet | $3–8 |
| Lightweight fast | GPT-4o-mini (cached), Haiku, Mistral 7B | $0.15–1.50 |

That's a 100–300x spread. Sending a "what's my account balance?" query to a $60/M model when a $0.50 model would nail it isn't just wasteful — in a regulated shop, it's a governance failure. Our compliance team treats unexplained AI spend the same way they treat unexplained cloud spend: as an audit finding waiting to happen.

### The cascade pattern

The pattern that works is a cascade. Think of it like a triage nurse, not a specialist.

1. **Semantic cache check** — Have we answered something semantically similar recently? Return the cached response. Zero cost, instant latency.
2. **Complexity classification** — Route the request. Simple lookup? Lightweight model. Multi-step reasoning with tool calls? Mid-tier. Genuinely novel architecture decision? Premium model.
3. **Escalation on failure** — If the cheaper model's output fails a quality gate (format validator, LLM-as-judge, deterministic check), retry with the next tier up.

This isn't theoretical. RouteLLM (LMSYS, ICLR 2025) showed 85% cost reduction on MT Bench routing between GPT-4 and Mixtral 8x7B while keeping 95% of GPT-4 quality. The BERT-based router variant runs in under 10ms — it adds effectively zero latency.

### What "complexity" actually means in practice

In our codebase, the classifier looks at a handful of signals:

- **Task type enum** — Classification, extraction, summarization, code generation, reasoning. We maintain a small lookup table mapped to model tiers.
- **Tool call budget** — If the agent needs >3 tool calls, it's probably not a simple lookup. Bump the tier.
- **Context size** — >16k tokens of retrieved context? Mid-tier minimum. The cheap models start losing the thread.
- **Regulatory flag** — PII, financial advice, compliance-critical paths. These get routed to models with approved data processing addendums, even if a cheaper model *could* do the task. Compliance-connected routing isn't optional in our world.

We started with a rule-based heuristic (query length + keyword signals). It worked well enough to prove the concept. Then we swapped in a fine-tuned BERT classifier trained on our own production traces labeled by outcome quality. The improvement was measurable: 12% more traffic correctly routed to the cheap tier without quality regression.

### The hidden cost of "just use the best model"

There's a second-order effect people miss. When every request hits the premium model, you lose the signal that tells you *which* requests actually need it.

If 90% of your traffic is simple classification and you route it all to GPT-4o, you have no data on whether a cheaper model would have sufficed. You can't optimize what you don't measure. The cascade forces you to instrument: every request gets a tier label, a latency, a token count, a cost, and a quality score. That telemetry becomes the training data for your next router iteration.

It also changes how you think about evals. You stop asking "does this model work?" and start asking "which tier handles this task type at acceptable quality?" That's a fundamentally different — and more useful — question.

### A concrete example from our stack

We have an internal "policy navigator" agent. HR, compliance, and engineering all ask it questions: "What's the travel policy for APAC?" "How do I request a security exception?" "Summarize the new data retention rules."

Initial version: every query → GPT-4o with a 50k token policy corpus in context. Cost per query: ~$0.40. Latency: 3–5 seconds.

After the cascade:
- Cache hit (exact or semantic match): ~$0.00, 200ms
- Simple lookup ("what's the travel per diem for Singapore?"): Haiku, ~$0.003, 800ms
- Multi-doc synthesis ("compare old vs new retention rules"): Sonnet, ~$0.08, 2.5s
- Novel interpretation ("does this clause cover crypto payments?"): Opus, ~$0.35, 4s

Blended cost dropped 87%. The expensive model still gets the genuinely hard stuff — but now it's *only* the hard stuff.

### The governance angle

In financial services, model routing isn't just an optimization. It's a control.

Our model risk management framework (SR 11-7 adjacent) requires us to document: which model processes which data, for what purpose, with what validation. A cascade architecture makes that documentation *easier*, not harder. Each tier has a model card, an approved use case list, and a data classification boundary. The router itself becomes a governed component — versioned, tested, auditable.

When the regulator asks "how do you ensure customer data doesn't hit an unapproved model?", the answer is a routing rule, not a hope.

### What's still hard

The cascade doesn't eliminate hard problems. It moves them.

- **Router drift** — As models update (and they update monthly now), the optimal routing decisions shift. You need continuous eval pipelines that catch regression before users do.
- **Edge case quality** — The 5% of queries where the cheap model *almost* works but subtly fails. Those are the ones that bite you in production. LLM-as-judge evaluators help, but they're not perfect — judge consistency is still a research problem as of late 2025.
- **Cold start latency** — Loading a router model (even a tiny BERT) adds milliseconds. For latency-critical paths, you warm the router or accept the overhead.

### The verdict

If you're running agents in production and every request hits your most expensive model, you're not "using the best tool." You're skipping the engineering discipline that makes AI sustainable.

Start with a three-tier cascade. Instrument everything. Let the data tell you where the quality cliff actually is. Your finance team will notice the difference before your users do.

And if you're in a regulated industry? The cascade isn't optional. It's the difference between "we have AI" and "we have AI we can explain to an examiner."