---
layout: post
title:  "Debugging on Autopilot - Who Is Still Learning the Codebase?"
date:   2026-07-04 10:15:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

The other week a flaky integration test in one of our Spring Boot services started failing on the build, and I did what I do these days: I pasted the stack trace into the coding agent and went to make coffee. By the time I was back, the fix was in — a test ordering dependency, one shared fixture leaking state between two test classes. Green build. Merged by lunch.

And here is the embarrassing part: if you asked me today exactly which fixture leaked what, I would have to look it up. The agent found the bug. I just carried the coffee.

<!--more-->

That small moment has been nagging at me, because debugging was always how I actually learned a codebase. Not reading the wiki, not the onboarding deck — getting stuck at 11 pm on a rolled-back transaction that looked like a dirty write, widening the log window, and finally seeing it. The bug was the teacher. Now somebody else — something else — attends the lesson on my behalf, and I sign the attendance sheet.

### The numbers are awkward

I am not the only one feeling this. Two studies from the last year put numbers on the unease, and both are worth reading in full.

The first is from METR, published last July. They ran a randomized trial with 16 experienced open-source developers doing 246 real tasks in repos they had contributed to for years, using early-2025 tooling — Cursor with Claude 3.5 and 3.7 Sonnet. Developers expected AI to make them 24% faster. After the tasks, they still believed it had made them 20% faster. The measured result: they were 19% slower with AI allowed. Feeling productive and being productive turned out to be two different things, and every one of us who has watched an agent confidently refactor the wrong method knows exactly how that happens.

The second one stung more. Anthropic published a study this January where developers had to pick up a new Python library, half with AI help and half without. The AI group finished a couple of minutes faster — a difference so small it was not statistically significant — but scored 50% on the follow-up quiz against 67% for the hand-coding group. Nearly two letter grades. And the biggest gap between the groups was on debugging questions. The people who used AI to get unstuck were the worst at explaining, afterwards, what had been wrong and why the fix worked.

Read that again, slowly. The skill that degrades fastest when the agent helps is the exact skill you need to check the agent's work.

### Comprehension debt

There is a name for the pile this leaves behind. A paper presented at EASE in Glasgow this April — based on 621 reflective diaries from 207 students over eight weeks — calls it comprehension debt: the gap between what a team knows about its codebase and what it needs to know to change it safely. The researchers found four ways it accumulates — accepting generated code as a black box, mismatched context the AI never had, slow atrophy from never tracing logic by hand, and skipping verification entirely — and only one way it gets paid down, which is using the tool as a tutor that explains rather than a contractor that delivers.

Their sharpest line, paraphrased: telling people to verify AI output is not enough, because verification is a competence, not a behavior. The people least equipped to spot the agent's mistake are the ones leaning on it hardest. In a regulated shop like mine, where an auditor can ask why a piece of financial logic changed, that sentence should be printed and pinned above every monitor.

### What I am trying instead

I am not giving up the agent — it genuinely saves me on boilerplate and on APIs I have never touched. But I have started keeping three small rules for debugging specifically, and they have changed how much I retain:

- I reproduce it myself first. Before the agent sees anything, I run the failing test, read the stack trace with my own eyes, and write down one sentence about what I think is happening. Often I am wrong. Being wrong on paper turns out to be an excellent teacher.
- I ask for explanations, not just fixes. The Anthropic study found the developers who asked conceptual questions and demanded explanations kept their understanding. So my prompt now usually ends with tell me why, and I actually read the answer instead of scrolling to the diff.
- I re-derive the fix before merging. If I cannot explain the change to a teammate in thirty seconds — what broke, why this fixes it, what else it could affect — the PR waits. This is my thirty-second code walkthrough, and it has caught at least two fixes that were green and wrong.

None of this is heroic. It slows me down on purpose, which feels absurd in a year when everything promises speed. But the METR result already showed me that the speed was partly imaginary anyway.

### The craft needs practice reps

Back in 2023 I wrote that debugging is a craft built on attention to detail and note-taking. I still believe that. What I would add now is that crafts need practice reps, and I had quietly outsourced all of mine. Nobody keeps their debugging eye sharp by reviewing diffs over coffee.

So here is my opinionated close: keep the agent, but stop letting it file the flight plan, fly the plane, and land it while you watch from seat 14A. Take the controls for the debugging parts, especially the confusing ones — those are the reps. The next outage will not ask who merged fastest. It will ask who understands the system well enough to fix it at midnight, with the agent down or the audit trail watching.

I want that person to still be me.
