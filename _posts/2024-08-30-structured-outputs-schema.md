---
layout: post
title:  "Structured Outputs - When Valid JSON Is Not Enough"
date:   2024-08-30 14:18:00 +0530
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

Three weeks ago OpenAI shipped [Structured Outputs](https://openai.com/index/introducing-structured-outputs-in-the-api/) on the API. Not another model name on a slide — a guarantee that the response will match a JSON Schema you supplied.

If you have been building tool loops or extraction pipelines this year, you already know the pain: the model returns *almost* the right shape, your parser throws, you retry, and somewhere a retry storm becomes the product. Valid JSON was never the hard part. The hard part was *your* shape.

<!--more-->

### JSON mode vs the thing we actually needed

JSON mode (DevDay 2023) was a useful half-step. It pushed the model toward braces and quotes. It did not promise that `customer_id` would be a string, that `status` would be one of three enums, or that a nested `conditions` array would even exist. You still wrote defensive parsers, fuzzy mappers, and "if key missing, ask again" glue.

Structured Outputs closes that gap in two places:

1. **Function calling / tools** — set `strict: true` on the tool definition. Available on models that already support tools, back through the June 2023 function-calling line (`gpt-4-0613`, `gpt-3.5-turbo-0613`, and later), including GPT-4o and GPT-4o mini.
2. **`response_format` with `json_schema`** — when you want a structured *answer*, not a tool call. Tied to the newer GPT-4o snapshots: `gpt-4o-2024-08-06` and `gpt-4o-mini-2024-07-18`.

On OpenAI's own complex-schema evals they report `gpt-4o-2024-08-06` with Structured Outputs at 100% schema match, versus under 40% for `gpt-4-0613` with prompting alone. Treat vendor eval numbers as directional, not gospel — but the direction matches what every production team has been screaming about for a year.

### What broke in my tool loops before this

I wrote in June that the agent is just a loop with better manners. The loop still is. The brittle joint was always step two: "validate args, then execute."

Concrete failures I kept hitting on internal POCs:

- **Almost-enums** — schema said `"pending" | "approved" | "rejected"`; model returned `"Pending"` or `"in_review"`. Downstream switch statement falls through; ticket lands in the wrong queue.
- **Optional-as-null theater** — field omitted entirely vs `"due_date": null` vs empty string. Three code paths for one business meaning.
- **Nested surprise** — tool expected `filters: [{ field, op, value }]`; model returned a single object or a free-text SQL fragment inside `value`.
- **Retry tax** — each failed parse meant another full prompt. Latency and cost stacked on the *failure* path, which is exactly when users already think the system is broken.

You can paper over that with Instructor, outlines, jsonformer, guidance — the open-source stack OpenAI even [acknowledges](https://openai.com/index/introducing-structured-outputs-in-the-api/) as inspiration. Client-side repair works until it does not. Server-side constrained decoding is a different contract: invalid tokens are masked out as the model samples, not cleaned up after the fact.

### How I am using it in a finance-shaped POC

Same posture as tool use generally: the model is a junior contractor with a badge, not a root shell.

- **Strict tools for anything that hits an API.** If the next hop is a real system of record, the args must deserialize into a typed object without a forgiveness layer. Pydantic on the Python side (or the equivalent on JVM) is the boundary; `strict: true` is what makes that boundary honest.
- **Structured answers for extraction, not for policy.** Meeting-note action items, form fields from a PDF dump, a ranked list of candidate accounts — great fit for `response_format.json_schema`. "Should we approve this credit exception?" is still a human decision with an audit trail, not a JSON field named `approved`.
- **Handle `refusal` explicitly.** OpenAI added a `refusal` field so a safety reject does not look like malformed schema output. In regulated workflows that distinction matters: refuse is a control working; parse error is your pipeline on fire.
- **Budget the first-schema tax.** First request with a new schema can be slow while they compile/cache the grammar (they say typical under ~10s, complex up to about a minute). Warm schemas in deploy or smoke tests; do not discover cold-start latency on a customer click.
- **Turn off parallel tool calls when you need the guarantee.** Structured Outputs is not compatible with parallel function calling matching every schema. Set `parallel_tool_calls: false` if you care more about shape than fan-out.

None of this replaces least-privilege credentials, human gates on side effects, or logging every tool span. Schema fidelity is necessary for a clean loop. It is not a control plane.

### What Structured Outputs does *not* fix

Worth saying out loud, because the blog posts will oversell it:

- **Wrong values inside a valid shape.** `"amount": 1000000` with the right type is still a bad transfer if the model hallucinated the digits. Validate business rules in *your* code.
- **Prompt injection and over-trust of retrieved text.** A perfectly typed tool call that exfiltrates data is still a breach. Schema is not authorization.
- **Vendor lock as architecture.** Constrained decoding on one API is great. Your domain types and policy checks should live in code you own, so a second model behind a thinner JSON contract does not mean rewriting the bank.

I still want deterministic checks after the model speaks. I just no longer want those checks to be "is this even JSON."

### The practical takeaway

Mid-2024 agent demos got louder. The useful engineering news in August is quieter: you can finally treat the model's outbound contract more like an OpenAPI response and less like a creative writing exercise.

If you already have a boring tool loop, the upgrade path is small and high leverage — flip `strict: true` on the tools that mutate or query real systems, move extraction prompts onto `json_schema`, and delete a layer of retry glue. If you do not have that loop yet, do not start with a multi-agent graph. Start with five tight schemas and a step counter.

Valid JSON was table stakes. Matching *your* schema is what lets the rest of the stack stay boring — and boring is what I want next to money, customers, and audit.
