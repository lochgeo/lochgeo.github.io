---
layout: post
title:  "ChatGPT - Three Weeks In and I'm Still Checking Everything"
date:   2022-12-23 10:34:00
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

OpenAI dropped [ChatGPT](https://openai.com/blog/chatgpt/) on 30 November as a free research preview, and by early December Sam Altman was saying it had crossed a million users. I signed up the first weekend, poked it with toy questions, then — because I am me — started feeding it the kind of work that actually lands on a .NET desk: failing unit tests, half-remembered LINQ, a nasty regex, a draft of an RFC nobody wanted to write.

Three weeks later I am impressed, slightly unsettled, and not ready to trust it with anything that ships without a human staring at the output.

<!--more-->

### What it is (and what it is not)

ChatGPT is a conversational interface on top of a GPT-3.5-series model, fine-tuned with reinforcement learning from human feedback — the same broad recipe as InstructGPT, tuned for dialogue. You type, it answers in paragraphs. You push back, it often revises. That loop is the product.

It is **not** GitHub Copilot with a chat window. Copilot lives in the editor and completes the file you are already in. ChatGPT lives in a browser tab and will cheerfully invent a whole mini-architecture if you let it. I wrote about [Copilot earlier this year](/GitHub-Copilot-My-Pair-Who-Never-Sleeps-And-Sometimes-Hallucinates/); the muscle memory is different. Copilot is Tab. ChatGPT is a meeting that never ends and never asks for a calendar invite.

There is also no "paste this into production" button. OpenAI is explicit that the preview can be wrong, biased, or overly confident. They are not wrong about that.

### Where it actually helped this month

The useful sessions looked like this:

- **Rubber-duck debugging with receipts.** I paste a failing xUnit assert and the method under test (sanitized), ask what could cause the null, and get three hypotheses ranked by likelihood. Two are usually noise. One is often the thing I was too close to see. I still open the debugger.
- **First drafts of dull prose.** Release notes, a short design note, an email explaining why we are not doing the clever thing. I rewrite half of it, but the blank page is gone.
- **Explaining code I already understand, worse.** "Explain this middleware pipeline like I am new to the team." Good for onboarding docs. Bad if you do not already know when it is lying.
- **Boilerplate shapes.** A sketch of a Polly retry policy, a Serilog enricher stub, a minimal API endpoint skeleton. Pattern completion at conversation speed. Same rule as Copilot: read every line.

What failed hard: anything that needed *our* domain. Internal status codes, the weird dual-write path we regret, "what does our auth handler do on token refresh." It invents plausible names and confident nonsense. Plausible is the dangerous part.

### The failure modes I keep hitting

OpenAI lists several of these in the launch post; I have now collected the scars myself.

- **Confident fiction.** It will cite a NuGet package version that does not exist, or describe an ASP.NET Core API the way it *used* to work two majors ago. Training cutoff is roughly 2021 knowledge for a lot of topics — fine for algorithms, rough for "what shipped last quarter."
- **Prompt brittleness.** Same question, slightly different wording, different answer. Sometimes it "does not know," sometimes it lectures. I treat every answer as one sample, not the truth.
- **Verbosity cosplaying competence.** Long answers feel authoritative. They are not. I have started asking for bullet points and "only facts you are sure about," which helps a little and does not fix the underlying problem.
- **Security theatre cuts both ways.** It refuses some harmful prompts and still falls for role-play jailbreaks people are posting everywhere. More relevant for me: it will happily draft insecure code patterns if you do not steer it. Humans do that too; ChatGPT just does it faster.

Stack Overflow temporarily banning ChatGPT-generated answers this month did not surprise me. A site built on verifiable answers cannot absorb fluent guessing at scale. Same instinct I want on our PRs.

### Rules I wrote on a sticky note

1. **No secrets in the box.** No customer data, no internal URLs, no connection strings, no unreleased designs. Public research preview means I treat the prompt like a postcard.
2. **Never ship unread output.** Code, config, or prose. If I cannot explain it in the PR description, it does not merge.
3. **Verify against primary sources.** Docs, source, a failing test turned green. ChatGPT is a hypothesis generator, not a system of record.
4. **Prefer small asks.** "Sketch a retry loop" beats "design our payments platform." Large answers hide large mistakes.

I am still a .NET person who lives in Visual Studio and VS Code. ChatGPT has not replaced either. It has replaced some of the time I used to spend staring at a blank buffer or fishing for the fifth blog post that almost matches my error message.

### What I am not doing yet

I am not putting this in front of customers. I am not wiring it into a support bot. I am not letting juniors treat it as a senior engineer who happens to type fast. Enterprise chatbots that sound sure and are wrong are a brand and compliance problem, not a demo win.

The interesting longer question is the interface, not only the model. GPT-3 in a playground was a curiosity for people willing to fight prompts. ChatGPT made the same family of models feel like a colleague. That packaging change is why your non-engineer friends are suddenly texting you screenshots. Implementation matters as much as parameters — a lesson product people have known forever, now arriving in AI with a bang.

### My take

Three weeks in, ChatGPT is the most useful unreliable tool on my desk. It accelerates the dull middle of writing and debugging. It will not own a design decision, a security boundary, or a production incident. Altman himself called it incredibly limited and a mistake to rely on for anything important right now. That lines up with what I have seen.

I am keeping the tab open. I am keeping the sticky note next to the monitor. And I am still the one who hits merge — because the model that talks back has never been on-call for my services, and until something changes about *truth*, that job stays human.
