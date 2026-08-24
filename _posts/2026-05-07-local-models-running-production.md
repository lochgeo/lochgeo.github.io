---
layout: post
title:  "Local Models - My Laptop Is Running Production Now"
date:   2026-05-07 08:12:00 +0530
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

One Saturday morning I opened Task Manager before I opened my mail. There it was: a model server holding nine-odd gigabytes of RAM, alive since Friday night, with a couple of things on the machine quietly pointing HTTP requests at it. A summarisation job had run overnight against my local model, on schedule, nobody watching. Somewhere in the past year or so, without any ceremony, my laptop stopped being a machine that occasionally plays with AI and became a small server that other software depends on.

I did not plan this. When I first ran models locally back in late 2023, it was a toy: Mistral 7B in a terminal, mostly to see whether it could hold a conversation at all. Fun toy. But a toy.

<!--more-->

### From chat window to endpoint

What changed is not really the models, it is where they sit. For the first year, running a local model meant opening a chat window, typing into it, marvelling, closing it. Then everything started speaking the same dialect: OpenAI-compatible HTTP endpoints. Editor extensions, note-taking tools, little Python scripts, agent harnesses, all of them just want a base URL and something shaped like an API key, and none of them care whether the answer comes from a hyperscale datacenter or from the i3 wheezing beside my knees.

This spring LM Studio went a step further and pulled its inference engine out into a headless command line tool, GUI entirely optional. Ollama has quietly run as a background service for ages. llama.cpp has had a server mode for the hands-on crowd forever. The direction of travel is unmistakable: the model is becoming infrastructure on our desks. And infrastructure has one iron law, learned by everybody eventually. The moment something depends on it, it is production. Even when the only something is you.

### The rules I stole from my day job

Once I admitted that to myself, a bunch of habits migrated over from work, where I spend my days keeping systems upright for other people:

- Pin versions. "Latest" is not a version. My nightly summariser points at a specific model file with a specific quantisation, written down in the script's README.
- Health check after every load. One known prompt whose shape I recognise. If the reply looks wrong, nothing downstream runs that night.
- Memory budgets. Sixteen gigabytes total means the model, the browser and the IDE are in a permanent three-way negotiation. Capacity planning did not stop being a real skill just because the datacenter shrank to a laptop.
- Keep prompt and response pairs for recurring jobs. Drift you cannot diff is drift you will never notice.
- One heavy consumer at a time. When the second tool hits the endpoint mid-job, tokens per second fall off a cliff and both callers sulk.

None of this is clever. All of it is the boring discipline that keeps production boring, which is precisely what production should be, applied here at a scale of one desk.

### Upgrades are dependency bumps

Gemma 4 landed in early April and, out of pure reflex from the demo days, I downloaded it that same week and pointed the summariser at it. Nothing crashed. That was the problem. The summaries came out structurally different, headers in new places, sections split differently, and a small script of mine that scrapes specific lines from the digest started missing them, silently, for days.

That is when it clicked properly: swapping a model file is a dependency upgrade, exactly like bumping a NuGet or Maven package. So the same rules now apply. Read the release notes. Canary it against one consumer first. Keep the old weights on disk until the new ones survive a full week of real jobs. I also keep a handful of golden prompts with eyeballed reference answers and run them after every swap before anything else gets flipped over. Disk space is cheap. Trust, once broken by a silent format change, is not.

### The 16 GB tax

Honesty requires the constraints paragraph. My laptop is a Core i3 with integrated graphics, no discrete GPU, and no NPU either; that wave of hardware arrived after this machine was already built. On paper, gpt-oss-20b fits inside sixteen gigabytes, which felt like a small miracle when OpenAI released the weights last August. In practice, "fits" and "usable" are different words on CPU-only inference: it loads, it reasons carefully, it writes beautifully and slowly, which makes it an overnight batch citizen rather than a conversation partner. Daily driving falls to the small quantised models, four to eight billion parameters, where speed is tolerable and quality stopped being embarrassing sometime in 2025.

The useful discipline this forces is deciding which jobs tolerate slow. Nightly summarisation, fine. Watching tokens crawl out one at a time while you wait, miserable. The cloud is still the right answer for interactive heavy lifting, and hybrid remains the honest architecture for most of us: anything sensitive stays on the laptop, everything else goes where the GPUs live.

If you run local models in 2026, my suggestion is simple. Stop treating them as apps and start treating them as a tiny production estate: pinned versions, health checks, rollback plans, and a maintenance window you actually respect. It sounds absurd to apply operations discipline to a laptop. It is also the difference between a demo that impresses guests and a tool your scripts can rely on at two in the morning. My laptop has an uptime number now. Frankly, it deserves a better dashboard than I give it, but then again, so does everything else I run.
