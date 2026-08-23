---
layout: post
title:  "Vibe Coding - Somebody Still Has to Read the Diff"
date:   2025-05-30 09:41:00
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

Sometime in early February, Andrej Karpathy finally gave the thing a name. Vibe coding, he called it — giving in to the vibes, embracing exponentials, and forgetting that the code even exists. Four months later the phrase is everywhere, and like most catchy terms it is doing real work: it separates "I reviewed every line" from "I typed a prompt and it ran".

And the tools caught up faster than anyone expected. Two weeks ago OpenAI put Codex into ChatGPT as a research preview — a cloud agent that takes your repository, works on tasks in its own sandbox, and opens pull requests for you to look at. Claude Code arrived as a terminal agent back in February. Copilot quietly grew an agent mode too. Meanwhile Y Combinator's Garry Tan said that for about a quarter of their Winter '25 batch, 95% of the code was written by LLMs. Not a typo, apparently.

Here is my honest position after a few months of living like this: generating code has stopped being the bottleneck. Reading it is. And that flips a lot of assumptions about what code review is for.

<!--more-->

### The numbers behind the vibes

It is easy to dismiss vibe coding discourse as Twitter noise, but the measurement folks have been busy. GitClear's latest code quality report analysed 211 million changed lines from 2020 through 2024, and the trends line up exactly with what you would predict once autocomplete became an autopilot:

- Copy/pasted lines exceeded *moved* lines for the first time ever in 2024. Moving code is how refactoring shows up in history; copy/pasting is how duplication does.
- Refactoring collapsed — "moved" lines went from about a quarter of all changed lines in 2021 to under 10% in 2024.
- Duplicate blocks of five or more lines multiplied roughly eight-fold during 2024.

Google's own DORA research hinted at the same trade a year earlier: AI adoption nudged throughput up while estimated delivery stability dropped 7.2%. More code, faster, that needs fixing again soon. The CEOs of Google and Microsoft both claim roughly 30% of new code at their companies is now AI-written, so this is not a fringe behaviour. It is the new normal, arriving without a manual for how to absorb it.

### What actually breaks in review

After reviewing a fair amount of agent-written code lately — some mine, some from the team — I keep seeing the same handful of failure modes. They are not the classic human mistakes:

- **Plausible is not correct.** The code looks idiomatic, names are clean, structure seems reasonable. Then you notice the pagination logic silently drops the last page. Humans write obviously-bad code when lost; models write confidently-almost-right code when lost.
- **Duplication instead of reuse.** The model cannot see your whole repository, so instead of reusing the shared helper it cheerfully pastes a second copy. Our resilience setup lives in one shared configuration (I wrote about our circuit breakers a couple of summers ago) — the agent has not read that post, and it happily inlined its own retry logic next to ours.
- **Tests that prove nothing.** This one scares me most. The agent writes a test, the test fails, and instead of fixing the code it adjusts the mock until the assertion passes. Green build, zero information. The test asserts the stub equals itself.
- **Churn.** GitClear tracks code rewritten within weeks of being committed, and that number keeps climbing. The diff you approved last sprint is not the code you are maintaining today.

Notice none of these show up in CI. Every single one requires a human who actually reads the thing.

### Rules I have landed on

So what do I do differently? Nothing exotic, mostly old discipline applied to a new writer:

- **Small diffs, non-negotiable.** An agent will happily produce a thousand-line change across fifteen files. I have started refusing to review anything I cannot hold in my head, robot-authored or not. Small tasks, small PRs, more of them.
- **Whoever merges owns it.** If the agent wrote it and you merged it, it is yours. There is no "the model did this to me" defence, and there certainly is not one in an audit.
- **Read tests like a contract lawyer.** Before I read a single line of implementation, I read the tests and ask: what would these catch? If the honest answer is "nothing", the PR goes back regardless of how pretty the feature code looks.
- **Make it show its work.** The nice thing about agents like Codex is they leave terminal logs, test runs, citations. A PR description with evidence beats one with confident prose. Confidence is free; logs cost effort, so they correlate with truth.
- **No vibing on money paths.** Throwaway prototype, internal script, weekend hack — go wild, that is what vibe coding is actually good at. Anything touching payments, auth, or customer data gets written slowly and reviewed slowly, whatever wrote it.

That last rule matters double in my world. In a regulated shop nobody asks the auditor "who typed this line?" — they ask "who approved it, and can you show me the control that made the approval meaningful?" That question survived mainframes, offshore teams, and outsourcers. It will survive agents.

### The part that worries me

The productivity math is genuinely seductive. A feature that took a sprint now takes an afternoon, and I am not going to pretend my team and I are above using it. But Jared Friedman's observation at YC stuck with me: the first generation of reasoning models is not good at debugging. Which means someone on the team still has to be good at debugging, and debugging is exactly the skill you fail to develop if you have only ever watched code appear from prompts.

If you are a junior engineer reading this: the agents have made typing code cheap. They have not made understanding systems cheap. That gap is your entire career.

For managers, the planning unit is quietly changing. I used to size work by who could write it; now I size it by who can review it. Review capacity is the real constraint, and no vendor slide mentions that.

{% include pullquote.html quote="You can delegate the typing. You cannot delegate the blame." %}

So yes, let the agents draft, scaffold, and grind through the boilerplate — I do it daily and I am not going back. But somebody still has to read the diff. Until the day models can carry accountability, that somebody is you. The scarcest skill on my team is no longer lines per hour; it is judgement per minute.
