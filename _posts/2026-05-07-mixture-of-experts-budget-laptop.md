---
layout: post
title:  "Mixture of Experts - My Old Laptop Runs Bigger Models Than It Should"
date:   2026-05-07 21:12:00
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

I have written about running LLMs on this laptop twice before. In late 2023 I got Mistral 7B humming away in llama.cpp and called it a glimpse of the future. Early last year the DeepSeek-R1 distills made reasoning models light enough for my hardware. Both times the punchline was the same though: local models were promising but compromised. You traded down to a smaller dense model, and you felt it in every single response.

This spring something changed, and it is not that my aging IdeaPad got faster. It did not. The models got sparser instead of smaller, and that turns out to be exactly the trade my hardware needed.

<!--more-->

### A Hospital Full of Specialists

Most models I used to run locally were *dense*: one generalist who has read every book in the library, and every question engages their entire brain. Ask them for a haiku and the same full brain lights up as when you ask for a distributed systems design.

A Mixture of Experts (MoE) model works more like a big hospital. There are many specialist departments, and a triage nurse at the door decides which few doctors actually see each patient. Mechanically:

- The feed-forward parts of the network are chopped into independent blocks called *experts*.
- A tiny router network scores all the experts for each incoming token and picks the top few.
- Only those selected experts do any work for that token. The rest stay asleep.

The catch is that the whole hospital still needs to be built and kept standing. All the expert weights live in RAM whether they get used or not. But the cost of each consultation is only a handful of specialists' time. You pay rent on everything, compute on almost nothing.

### Sparse Beats Small on Old Iron

Here is the rule of thumb that makes this matter for laptops: generating a token is mostly the CPU reading weights out of memory, so your tokens-per-second is roughly memory bandwidth divided by the bytes you read per token. For a dense model that means *all* the weights, every time. For a sparse model it means only the awake experts.

Which gives you two separate bills:

- **Total parameters** decide your RAM bill. Everything must fit.
- **Active parameters** decide your speed bill. Only those get read per token.

The open-weight world has been marching down this road for a while now. Mixtral teased it in December 2023 with two of eight experts active per token. Then April 2025 brought Qwen3-30B-A3B: 30.5 billion parameters in total, of which only about 3.3 billion wake up per token, drawn from 128 experts picking eight at a time, under an Apache 2.0 license. And in August 2025 OpenAI itself went back to open weights for the first time since GPT-2, with the gpt-oss family. The smaller one, gpt-oss-20b, is 21 billion parameters with 3.6 billion active, shipped natively quantized to MXFP4 so the checkpoint lands around thirteen gigabytes and officially runs within 16 GB of memory. Twenty-one billion parameters, on the same laptop that struggled with seven dense ones.

One honesty note before you run off to download the biggest sparse thing you can find: sparsity saves compute per token, not the memory needed to hold the weights. Llama 4 Scout has only 17 billion active parameters, but around 109 billion total ones, and its quantized weights weigh in north of fifty gigabytes. That is a workstation model no matter what the active number whispers at you. On a 16 GB machine like mine, the sweet spot is precisely this class: twenty-ish billion total, three-ish billion awake.

### What I Actually Run

My setup has not gotten exotic. LM Studio for the GUI and the server, Ollama when I am living in the terminal. LM Studio became an MCP host back in June 2025, dropped its "not for work" restriction a month later, and the 0.4 release this January finally made the server side feel grown up rather than bolted on.

But the feature that actually changed my workflow is barely a feature: an OpenAI-compatible endpoint sitting on `localhost:1234/v1`. My scripts genuinely cannot tell the difference between that and a hosted API. One environment variable decides where my tokens come from:

```python
client = OpenAI(base_url=os.environ["LLM_BASE_URL"])
```

Point it at the cloud when the task deserves the frontier. Point it at localhost when it does not. Same code path either way.

### Where It Fits a Banker's Week

Working in finance, this is where local models stopped being a hobby and started being useful:

- **I paste things I would never paste into a cloud endpoint.** Production incident logs that contain customer identifiers, half-finished risk reports, chunks of vendor contracts. Nothing leaves the machine, so there is no compliance ticket to raise and no awkward conversation afterwards.
- **Boilerplate at zero marginal cost.** Test scaffolding, regexes I always half-forget, commit messages, throwaway rename scripts. No rate limits, no mental meter ticking per request.
- **Airplane-mode work.** Long flights and dodgy hotel Wi-Fi used to mean no assistant at all. Now the assistant is the one thing that definitely works offline.

The frontier APIs still get the hard stuff: design reviews, genuinely gnarly debugging, anything where being wrong is expensive. Deciding which tasks belong to which tier is a skill in itself, and I suspect it becomes a core developer skill the way choosing the right data structure once was.

### What It Still Cannot Do

Long agentic sessions eat RAM for breakfast; every turn of context sits in memory next to the model, and my 16 GB ceiling means Windows and I negotiate constantly. The very hardest reasoning still belongs to the big hosted models. And there is a knowledge cutoff with no web access, so anything recent means a trip to the browser anyway.

Still, my verdict is settled: sometime in the past year, without a launch event, on-device inference crossed over from party trick to infrastructure. Give it a couple of years and "AI laptop" will be as meaningless a marketing badge as "HD-ready" was. The interesting question is no longer whether your machine can run a serious model. It is which parts of your week belong to it. Mine, apparently, is anything I would be embarrassed to paste into somebody else's datacenter.
