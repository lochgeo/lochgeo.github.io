---
layout: post
title:  "Local LLMs - The Model That Never Leaves My Laptop"
date:   2023-11-30 14:27:00
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

OpenAI's DevDay was three weeks ago. GPT-4 Turbo with a 128k context window, cheaper tokens, custom GPTs, an Assistants API with built-in retrieval. The cloud path just got louder and, for once, cheaper — Turbo input is about $0.01 per 1k tokens and output about $0.03, roughly a third and half of what GPT-4 was charging.

I still spent last weekend trying to get a 7B model to answer questions *without* leaving my machine. Not because I think my laptop beats a frontier API. Because in a regulated shop, "the prompt never left the building" is sometimes the only sentence that gets a POC past the first security review.

<!--more-->

### What "local" actually means right now

A year ago local meant toys and research dumps. Late 2023 it means a real toolchain:

- **[llama.cpp](https://github.com/ggerganov/llama.cpp)** — Georgi Gerganov's C/C++ inference stack that made CPU (and light GPU) runs of Llama-class models feel normal. Started in March after Meta's LLaMA weights leaked into the wild; it is still the gravity well everything else orbits.
- **GGUF** — the single-file model format that landed in llama.cpp in August. Metadata and tensors in one package, built so loaders stop breaking every time someone adds an architecture. If you download a quantized model from Hugging Face this month, it is probably a `.gguf`.
- **Mistral 7B** — released end of September under Apache 2.0. Small enough to fit on a modest box after 4-bit quantization, strong enough that people stopped treating 7B as a joke. Dense model, no mixture-of-experts magic required.
- **Ollama** — thin wrapper that makes "pull a model, run a chat, hit a local HTTP endpoint" feel like Docker for weights. Preview builds showed up mid-year; by autumn it is how a lot of us stop fighting compile flags on a weeknight.

I am not running a data-center GPU. Think weekend-lab hardware: a normal Windows laptop, limited RAM, Docker already installed for other experiments. The point is not tokens-per-second bragging rights. The point is a loop I control end to end.

### Why I bother when Turbo got cheaper

Cloud models won the quality war for general work. That is not the argument.

The argument at work looks more like this:

- **Data residency is not a slide.** Customer text, internal runbooks, half-finished policy docs — some of that cannot go to a third-party API on day one of a POC, no matter how friendly the retention language got this year.
- **Air-gapped demos beat slide decks.** Showing a stakeholder a chat that works on a machine with the Wi-Fi off lands differently than "we will configure VPC endpoints later."
- **Cost is predictable in a different way.** API bills scale with curiosity. A local model scales with electricity and your patience. For spike work and eval loops, that is sometimes cheaper *and* calmer.
- **You learn the failure modes.** Quantization artifacts, short context, weak instruction following — you feel them in your hands. That makes you a better buyer of the big APIs, not a worse one.

I still use the hosted APIs for real writing help and for anything that needs broad world knowledge. Local is the lab bench, not the factory floor.

### The first honest experiment

My bar for "it worked" is boring on purpose:

1. Pull a small instruct model (Mistral 7B class, Q4 quantization).
2. Ask it something I already know the answer to from our domain *phrased as a junior would ask*.
3. Ask it something that requires a doc that is *not* in the weights — then wire a tiny retrieval step over a folder of markdown.
4. Write down where it lied with confidence.

Without retrieval, local 7B models are fine for drafting, rewriting, and "explain this stack trace like I am tired." They are unreliable narrators about *your* systems. Same lesson as the ChatGPT tab in December, just slower and offline.

With a dumb RAG loop — chunk markdown, embed, stuff top-k chunks into the prompt — you get something more interesting and more fragile. Chunk boundaries matter. Overlap is a tradeoff, not a free lunch. The embedding model and the generator do not share a brain; they only share your glue code. None of that is new theory. It just hits harder when the whole pipeline is on one laptop and you cannot blame "the platform."

### What I would not do with this yet

- **Customer-facing answers.** No. Quality and safety bars are not met by a weekend GGUF and hope.
- **Unreviewed code merges.** Local coding help is a faster rubber duck. It is not a reviewer. Same discipline as Copilot last year: read every line.
- **Skipping the cloud conversation entirely.** Legal and security still need a written path for when you *do* need GPT-4-class reasoning. Local POCs buy you time and evidence; they do not retire the vendor review.
- **Pretending 7B replaces Turbo.** 128k context and stronger reasoning still live in the API for a reason. Use the right hammer.

### How this changes the backlog conversation

Before DevDay, "we need ChatGPT" meant a browser and a policy headache. After DevDay it means Assistants, retrieval add-ons, and a store full of custom GPTs that product folks will screenshot into Slack. Before you green-light any of that against real data, a local spike answers cheaper questions:

- Can we even phrase our docs so a model can use them?
- Which questions are retrieval problems vs generation problems?
- What does a refusal look like when the docs do not support an answer?

Those are engineering questions. You can work them on a plane without a VPN.

### Closing

The useful split in late 2023 is not "local vs OpenAI." It is **where does the data go on the first prototype**, and **which failure mode am I debugging — model quality, retrieval quality, or process**. llama.cpp and friends made the first question something you can answer on hardware you already own. DevDay made the second path cheaper and more productized.

I will keep a small model on the laptop. I will keep an API key for the hard problems. And I will keep treating both like junior teammates who sound sure even when they are guessing — because that part did not change when the weights started fitting in RAM.
