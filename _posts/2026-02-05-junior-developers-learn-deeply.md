---
layout: post
title:  "Learn It Deeply - The Skills Autocomplete Cannot Give You"
date:   2026-02-05 10:15:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

It is hiring season again, and I have been reading a stack of CVs that all say some version of the same thing: "proficient in AI-assisted development". In interviews, candidates live-code impressively fast. The autocomplete hums, the boilerplate appears, the endpoint compiles. Then I ask them to trace a null pointer through three layers of a Spring service they did not write, and the room goes quiet.

I am not complaining about the tools. I use them every day and I am not giving them up. But somewhere between the demos and the hiring loops, we seem to have confused typing code with understanding systems. Those were never the same skill, and the gap between them is now the whole game.

The numbers back up the unease. Last summer METR ran a randomized trial with sixteen experienced open-source developers on their own mature repositories, mostly using Cursor with Claude 3.5 and 3.7 Sonnet. Developers expected a 24% speedup. They took 19% longer with AI enabled — and afterwards still believed the tools had made them 20% faster. They accepted less than 44% of the generated code untouched. And in last year's Stack Overflow survey of over 49,000 developers, the top frustration, cited by 66%, was "AI solutions that are almost right, but not quite", with 45% saying debugging AI-generated code takes more time. Usage keeps climbing while trust keeps falling.

<!--more-->

### Typing was never the job

Here is the uncomfortable reading of those studies: the bottleneck was never how fast you can produce lines. It is how fast you can decide which lines are right. The autocomplete made the cheap part cheaper and left the expensive part — judgement — exactly where it was. A junior who leans on the tool for the first and never trains the second ends up fast at producing code they cannot evaluate. That is not leverage. That is a loan with interest, payable the first time production breaks at 2 AM.

Think of it like learning to drive with a car that parks itself. Wonderful feature. But if you never learned to judge distances yourself, you are not a driver with assistance — you are a passenger with opinions.

### The short list I would learn deeply

If you are early in your career and wondering what survives the autocomplete, here is my list. None of it is glamorous. All of it compounds:

- **Reading code you did not write.** You will read ten times more code than you ever type. Practice following a request from controller to database through code you have never seen, without running it first. The developers who can do this cold are the ones everyone wants on their team, AI era or not.
- **Debugging as a discipline, not a vibe.** Form a hypothesis, design the smallest experiment that could disprove it, widen the log window, check the assumption you are most sure about first. I once spent a day chasing a dirty write that turned out to be a rolled-back transaction — found only because I stopped trusting my mental model and read what the logs actually said. No tool does that part for you.
- **How the machine runs your code.** HTTP semantics, SQL beyond the ORM, threads and connection pools, what happens between "deploy" and "serving traffic". Abstractions leak on schedule, and the person who knows what is underneath the leak is the person who fixes it.
- **Git forensics.** Bisect, blame, reading a diff like a story. When something broke three sprints ago, the history is the witness statement. Learn to interrogate it.
- **Testing as specification.** A test says what the code should do, in executable form. Anyone can generate assertions; few can decide what is worth asserting. That judgement is the skill.
- **One boring technology, deeply.** Pick the database, the framework, the messaging layer you touch daily and learn it past the tutorial line. Depth in one place teaches you how depth works everywhere.

### How to actually build those muscles

Advice without a practice plan is just decoration, so here is what I tell juniors on my team. First, write the first version yourself more often than feels efficient — the struggle is the learning, and the tool will still be there for the second draft. Second, fix one real bug a month with the assistant switched off, start to finish; treat it like going to the gym. Third, read every diff you submit as if a sceptical colleague wrote it, because one day an auditor will read it exactly that way. Fourth, explain things out loud — to a teammate, to a rubber duck, to a blank page. The survey found 61% of developers still ask a person because they want to fully understand their code. Understanding is not a side effect of shipping; it is the job.

In my world there is an extra edge to this. In a regulated shop, nobody at the review board asks which tool typed the line. They ask who understood it, who approved it, and where the control is that made that approval meaningful. Accountability does not autocomplete. The signature at the bottom is still yours, and signatures require comprehension.

{% include pullquote.html quote="Autocomplete made typing cheap. It did nothing to the price of understanding." %}

So yes, keep the assistant open — I do. Let it scaffold, translate, and grind through the boilerplate. But learn the fundamentals on purpose, while the stakes are still low and someone senior is still around to check your work. The developers who will be valuable five years from now are not the ones who type fastest today. They are the ones who know what the code does, why it does it, and what to do when it lies to them at 2 AM.
