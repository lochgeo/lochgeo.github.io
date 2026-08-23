---
layout: post
title:  "ChatGPT API - Yesterday the Floodgates Opened"
date:   2023-03-02 19:22:00
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

Yesterday OpenAI shipped what a lot of us have been waiting for since late November: a real [ChatGPT API](https://openai.com/blog/introducing-chatgpt-and-whisper-apis). Not a scraped session cookie. Not another playground experiment. A `chat/completions` endpoint, a model called `gpt-3.5-turbo`, and a price that makes side projects feel cheap again — $0.002 per 1k tokens, which they say is about 10x cheaper than the older GPT-3.5 completions path.

I spent December treating the browser tab like a slightly unhinged junior. I wrote about that [three weeks in](/ChatGPT-Three-Weeks-In-and-Im-Still-Checking-Everything/). Yesterday moves the same model family from "interesting tab" to "thing you can put behind a service boundary." That is a different conversation at work.

<!--more-->

### What actually shipped

The useful bits, stripped of the launch-day noise:

- **`gpt-3.5-turbo`** — same family as the ChatGPT product, exposed as a first-class API model. Snapshot `gpt-3.5-turbo-0301` is the pinned version if you do not want silent upgrades.
- **Messages, not a single prompt blob.** You send a list of roles (`system`, `user`, `assistant`) instead of stuffing history into one string. Under the hood it is still tokens; the contract is just closer to how people already talk to the product.
- **Whisper on the API too** — speech-to-text at $0.006 per minute, on top of the open-source model they released last September. Handy if you do not want to babysit GPU boxes for transcription.
- **Policy changes that matter for enterprise nerves** — API data is not used to improve models unless you opt in; default retention is 30 days, with stricter options if you need them. Users own inputs and outputs. Pre-launch review is gone.
- **Dedicated capacity** — reserved compute if you are past hobby volume (they ballpark ~450M tokens/day as the economic crossover). Shared multi-tenant is still the default.

Early logos on the blog — Snap's My AI, Quizlet's Q-Chat, Instacart, Shopify's Shop app, Speak — are consumer product demos. Fine. The interesting question for me is not "can we build a haiku bot." It is "can we run a narrow internal POC without lighting compliance on fire."

### What changes from the browser tab

In December the failure mode was personal: I pasted too much into a research preview and had to trust a sticky note. An API changes the shape of the mistake.

- **You now own the prompt plumbing.** System message, history window, retries, timeouts, logging. The model stops being a website and becomes a dependency with latency and a bill.
- **You can put a thin service in front of it.** Auth, rate limits, allow-lists of who may call it, redaction before the request leaves your network. That was awkward when the "client" was every engineer with a ChatGPT account.
- **Cost becomes a line item, not a free preview.** At two-tenths of a cent per thousand tokens, a team can burn money quietly with chatty retries and giant context dumps. Meter it on day one or explain the invoice later.
- **Versioning is your problem.** Pin `gpt-3.5-turbo-0301` if behaviour must stay still. Floating `gpt-3.5-turbo` will move under you — OpenAI already said they will roll the alias forward. Same lesson as any SaaS SDK: pin what you test.

I still live in .NET most days. Calling this from a small ASP.NET Core worker or a Function is boring HTTP — which is exactly why every product manager with a slide deck will suddenly want a "ChatGPT feature" in the backlog by Friday.

### The POC I would actually green-light

Not a customer-facing agent. Not "replace search." Something dull and measurable:

1. **Internal first-draft only.** Release notes from a ticket list. A summary of a long RFC for people who will not read it. A suggested reply the human must edit. Output never ships unread.
2. **No production data in the prompt.** Synthetic or public docs until legal and security have a written position. The opt-out and retention changes help the conversation; they do not replace a data classification review.
3. **One endpoint, one team, one metric.** Acceptance rate of drafts, time saved on a known chore, or "how often did a human throw the answer away." If you cannot measure it, it is a demo, not a POC.
4. **Hard stop on autonomy.** No tool calling into write paths. No "the bot filed the ticket." Read-only helpers first. Write paths wait until you trust the failure modes.

The pattern that failed me in the browser still fails over HTTP: confident fiction about *our* domain. Internal status codes, half-documented services, the auth handler nobody fully likes — the model will invent a plausible story. An API just lets that story scale to every caller you authorize.

### Guardrails before the hackathon posters go up

A few things I want written down before anyone merges an integration:

- **Secrets stay out.** Keys in a vault, not in a notebook. Prompts treated like logs that might leave the building.
- **Human in the loop is not optional for anything customer-visible.** Fluency is not correctness. We already learned that with Copilot and with the December tab.
- **Eval a golden set.** Ten prompts with known-good answers beat "it felt smart in the demo." When the model alias moves in April, you will want the regression.
- **Budget and kill switch.** Per-team token caps. A feature flag that turns the whole path off without a deploy drama.
- **Uptime expectations stay honest.** OpenAI themselves admitted the last two months of API uptime were not where they wanted. Design for degradation — queue, retry with backoff, fall back to "write it yourself."

### My take

Yesterday did not make the model trustworthy. It made the model *callable*. That is enough to start serious internal experiments and not enough to hand the keys to a support bot that talks to customers.

If your org has been stuck on "we cannot put ChatGPT in the architecture because it is a website," that excuse expired on 1 March. The new excuses are better ones: data handling, evaluation, cost control, and whether the use case is actually a language problem or just a missing search index with better marketing.

I am going to wire a tiny .NET console against `chat/completions` this weekend, pin the 0301 snapshot, and try the same dull tasks I already do in the browser — with logging and a hard rule that nothing from the response lands in git without me reading it. The floodgates opened. I still want a gate on my side of the river.
