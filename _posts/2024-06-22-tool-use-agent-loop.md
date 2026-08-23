---
layout: post
title:  "Tool Use - The Agent Is Just a Loop With Better Manners"
date:   2024-06-22 11:42:00
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

OpenAI shipped GPT-4o in mid-May. Anthropic took tool use generally available across the Claude 3 family at the end of May. Every internal slide deck I have seen since then has the word *agent* on it twice.

Most of those decks are still describing a chatbot with extra steps. The useful shape is simpler and less glamorous: a model that emits a structured tool call, your code that runs it, a result that goes back into the conversation, and a hard stop when the loop has gone on long enough.

<!--more-->

### What actually changed this spring

Function calling is not new. OpenAI put it on the API in June 2023. What mid-2024 feels like is *table stakes* — the same primitive on every frontier model you would actually put near a bank workflow, with better schema adherence and fewer "I will call the tool in prose" failures.

GPT-4o ([Hello GPT-4o](https://openai.com/index/hello-gpt-4o/), 13 May 2024) is the model people are pointing the loop at right now: faster and cheaper than GPT-4 Turbo for a lot of tool traffic, solid at picking among a small tool list, still perfectly capable of inventing an argument shape you did not define. Claude's tool use going GA on 30 May 2024 ([Anthropic's announcement](https://www.anthropic.com/news/tool-use-ga)) matters for the shops already on Bedrock or Vertex — same idea, different JSON envelope, your application still holds the keys.

The demo people clap for is "the assistant booked the meeting." The engineering object underneath is still:

1. Send messages + tool definitions.
2. If the model returns a tool call, validate args, execute *your* function, append the result.
3. Call the model again.
4. Repeat until it answers in plain text — or you hit `max_steps`.

That is the whole product. Frameworks are optional sugar around that loop.

### The loop is the product, not the model

I keep watching teams burn a sprint wiring LangChain / LlamaIndex / "multi-agent orchestration" before they have one boring path that works:

- One model.
- Three to seven tools with tight JSON schemas.
- A `while` with a step counter.
- Logging of every tool name, args, latency, and outcome.

Until that path is boring, a multi-agent graph is cosplay. You cannot debug a swarm if you cannot debug a single ReAct-style turn (the 2022 pattern is still the mental model: thought → action → observation, even when the "thought" is mostly internal to the model and the "action" is a native tool call).

Concrete failure modes I have already hit on internal POCs:

- **Tool soup** — fifteen overlapping tools; the model picks the almost-right one and you spend the afternoon reading traces.
- **God tool** — `run_sql(query: string)` with the service account that can see everything. Prompt injection becomes data exfiltration with one cooperative model turn.
- **No max steps** — a confused model retries the same lookup until the token bill looks like a cloud cost incident.
- **Trusting the model's story of what it did** — the UI says "I updated the ticket"; your audit log is empty because you never executed a tool, you only rendered assistant text.

### How I am wiring tools in a regulated shop

Treat the model like a junior contractor with a badge that opens three doors, not the whole floor.

- **Least privilege per tool.** Separate credentials. Read-only search is not the same principal as "create ticket." No "whatever the logged-in user can do."
- **Human gate on side effects.** Draft the email, open the change request, propose the entitlement change — a human (or a deterministic policy engine) confirms in *your* UI. The model does not get a send button.
- **Validate before execute.** JSON Schema on the way in is necessary and not sufficient. Check enums, ID formats, tenant scope, and "does this customer_id belong to the caller" in code. Reject loud; return a structured error the model can recover from once, not a stack trace dump.
- **Small tool menus.** If the workflow has phases (identify customer → gather facts → propose action), swap the tool list per phase. Fewer choices, fewer creative mis-fires. People reinvent this every week and call it "routing."
- **Idempotency and timeouts.** Tools get a deadline. Writes get an idempotency key. A retried loop must not double-book a payment or double-file a case.
- **Log the trace like a distributed transaction.** Correlation ID across user turn, model request IDs, tool spans. When compliance asks "why did it touch that record," you need a chain, not a vibe.

None of that is model magic. It is the same discipline we already use for any service that can mutate state — applied to a nondeterministic caller.

### What I am not doing in June 2024

- Shipping an unsupervised agent that can move money, change access, or mail clients because "GPT-4o is better at tools now." Better is not a control.
- Letting the framework own secrets and HTTP. If I cannot point at the line that performs the side effect, I do not understand the blast radius.
- Stuffing the system prompt with novel-length policy and calling it governance. Policy belongs in code paths and review queues; the prompt is guidance.
- Waiting for perfect schema adherence. Parse, validate, reject. The model will still surprise you on a Tuesday.

### The practical takeaway

Agent hype right now is loud because the primitives finally feel production-shaped on more than one vendor. The durable skill is not picking a framework logo. It is designing a tiny, observable loop with mean tools and boring guardrails.

If your mid-2024 backlog says "build an agent," translate it to: **define five tools, write the loop, log every step, put a human on anything that leaves a mark.** Ship that. Measure where it fails. Then decide whether you earned a second agent.

I would rather have one tool loop that audit can explain than a slide full of autonomous boxes that nobody can stop mid-flight.
