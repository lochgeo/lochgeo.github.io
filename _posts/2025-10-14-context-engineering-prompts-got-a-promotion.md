---
layout: post
title:  "Context Engineering - Prompts Got a Promotion"
date:   2025-10-14 09:41:00 +0530
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

Every few months this field renames something and half of us roll our eyes. This time I think the rename earns its keep. In late June, Andrej Karpathy came out in favor of "context engineering" over "prompt engineering", describing it as "the delicate art and science of filling the context window with just the right information for the next step". Shopify's CEO Tobi Lütke had been pushing the same term as a core skill, Simon Willison endorsed it within days, and by the end of September Anthropic had shipped an entire engineering guide called [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents). When that many practitioners converge inside one summer, a word has usually caught up with reality.

Here is my confession. I have spent much of the past year building chatbots and RAG pipelines at work, and a good share of my "prompt engineering" sessions were me fiddling with wording while the real problem sat somewhere else entirely: stale chat history, retrieved chunks nobody asked for, tool output dumped three turns ago still squatting in the window. The prompt was never the whole system. Naming the rest of it is oddly liberating.

<!--more-->

### Why the rename matters

Willison defined it as the art of providing all the context for a task to be *plausibly solvable* by the model. Notice what that includes, from Karpathy's own list: task descriptions, few-shot examples, RAG, related data, tools, state and history, compaction. That is not typing skills. That is system design.

The old term had a branding problem too. As Willison put it, most people's inferred definition of prompt engineering was "a laughably pretentious term for typing things into a chatbot". Meanwhile anyone running an LLM app in production knows the truth is closer to what Karpathy said when dismissing the "ChatGPT wrapper" sneer: an LLM app is a thick layer of software coordinating many calls, and context is its central resource. The new name describes the actual job, which is why I expect this one to stick.

### The window lies to you

The uncomfortable research behind all this: bigger windows did not make filling them safe.

A [Chroma technical report](https://research.trychroma.com/context-rot) from July tested 18 models, including the frontier ones, and found performance degrading as input length grows, even on trivially simple tasks like retrieval and repeating a sequence of words. Distractors, passages related to the question but wrong, made things worse in uneven ways. Their conclusion was blunt: models do not process context uniformly, so curating what goes in matters more than maximizing how much fits.

Then there is the [multi-turn study](https://arxiv.org/abs/2505.06120) from Microsoft Research and Salesforce in May. They took the same instructions and either gave them fully up front or dribbled them out across conversation turns, like real users do. Single-turn performance averaged around 90%; sharded across turns it dropped to roughly 65%, a 39% relative hit, with unreliability more than doubling. Their one-line summary deserves a frame: when LLMs take a wrong turn in a conversation, they get lost and do not recover.

I felt vindicated reading both. Last quarter our support chatbot got worse after we started appending the full conversation history every turn, and nobody could explain why. Turns out we were reproducing these experiments on our own users. More context was literally the bug.

I think of the context window like the kitchen counter now. A bigger counter does not make you cook faster. It mostly gives you more places to lose the spatula.

### What it looks like day to day

So what does one actually *do*? Anthropic's guide frames it as managing a limited attention budget, finding the smallest set of high-signal tokens that gets the outcome you want. The concrete levers:

- **Just-in-time retrieval.** Stop pre-loading everything. Let the agent grep, tail and read files on demand instead of stuffing whole documents in up front.
- **Compaction.** When the conversation nears the limit, summarize it and restart from the summary. Claude Code does this: it keeps architectural decisions, unresolved bugs and implementation details, throws away redundant tool output, and continues with the compressed context plus the five most recently touched files.
- **Structured note-taking.** Have the agent persist notes outside the window, a NOTES.md or todo list it writes and re-reads later. Cheap external memory, very little machinery.
- **Sub-agents.** Give focused subtasks their own clean windows; each can burn tens of thousands of tokens exploring but returns only a distilled 1,000–2,000 token summary to the coordinator.

None of these are exotic. All of them are plumbing decisions you make once and live with, which is exactly what "engineering" is supposed to mean.

### Evals or it didn't happen

Here is the part I care about most, because it separates discipline from vibes. You cannot iterate on context if you cannot measure the result. Every lever above changes behavior in messy, non-obvious ways, and eyeballing two sample conversations tells you nothing.

Hamel Husain [wrote this plainly](https://hamel.dev/blog/posts/evals/) back in early 2024: failed LLM products almost always share one root cause, no robust evaluation system. His ladder is still the right one, unit-test assertions on outputs, then human and model-based review of traces, then A/B tests, cheapest level run on every change.

We stumbled into this at work the honest way, by getting burned. After the history-append regression I mentioned, we finally built a small golden set, about thirty questions pulled from real support transcripts with expected answers agreed by the domain expert. It runs after every change to prompts, chunking, or history policy. It is maybe two hundred lines of Python and it has caught more silent regressions than any code review ever has. Even Anthropic's own advice assumes this loop exists: they suggest tuning your compaction prompt by maximizing recall first, then precision, against real agent traces. You cannot do that without a pile of labeled examples.

The regulated-shop bonus is familiar from everything else in this space: a versioned eval suite turns "the model seems better" into a diff someone can review and an auditor can read.

### The promotion is deserved

My verdict: this rename is not hype cycling, it is scope correction. Prompt engineering was always a subsystem of something bigger, and pretending otherwise made teams tune wording while the architecture rotted underneath. Treat context like any other managed resource, with ownership, budgets, and tests.

And a prediction, cheap to make in October 2025: the windows will keep growing, and it will not matter. The skill that compounds is deciding what deserves to be in front of the model at step twelve, not how many tokens fit. Curate the counter, measure everything, and let the marketing numbers fight about millions of tokens without you.
