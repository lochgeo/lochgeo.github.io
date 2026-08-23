---
layout: post
title:  "Background Agents - When the Backlog Opens Its Own Pull Requests"
date:   2026-02-28 09:12:00
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

Last sprint I ran a small experiment. Instead of assigning a routine bug ticket to a teammate, I assigned it to our coding agent. Twenty minutes later, while I was still in standup, a draft pull request opened itself: commits pushed, checks running, a tidy checklist of what it intended to do. Nobody typed anything. The backlog had effectively started opening its own pull requests.

I have written before about reading AI-generated diffs and about repo instructions for coding agents, but those were about the inner loop, me at the keyboard with an agent beside me. Background agents are a different animal. They change not how I code, but how work flows through the whole team. Somewhere along the way my issue tracker quietly turned into a job queue.

<!--more-->

### Same skeleton under every logo

When I compared notes with friends who use other tools, the funny part was how little there was to compare. Every serious player in this space converged on the same shape: you hand over a task, the service boots an isolated virtual machine or container, clones your repo, edits files, runs your test suite, iterates until things go green, opens a pull request, and stops. Devin started this dance back in 2024. Then 2025 happened in a rush: OpenAI's Codex cloud agent arrived in May, Cursor shipped background agents around the same time, Google's Jules reached general availability in August, GitHub made the Copilot coding agent generally available in September, and Anthropic put Claude Code on the web in October with git credentials deliberately kept out of the sandbox behind a proxy. Roughly six months, five vendors, one architecture.

What differs is mostly the doorway. GitHub wants you to assign an issue, exactly like you would assign a junior developer. Devin and Cursor live in Slack, waiting for an @-mention. Jules lives in the browser, Codex lives in ChatGPT. Pick whichever doorway matches where your team already works; underneath, it is the same clone-edit-test-open-PR loop every time.

### What we actually delegate

After a couple of months of feeding the queue, our unwritten policy looks like this. Good candidates:

- Test coverage gaps, where "done" is measurable in percentage points
- Dependency bumps and the small breakages they cause, provided CI catches them
- Bug fixes where we write the failing test first and let the agent make it pass
- Documentation drift, lint debt, dead code removal

Things we do not delegate, and I do not see changing soon:

- Authentication, authorization, anything session-related
- Any code path that moves or transforms customer money
- Schema migrations on shared databases
- Any ticket whose acceptance criteria we cannot state precisely

That second list is really one rule: if we cannot describe what correct looks like, the agent will happily invent its own version of correct. In a regulated shop, inventing correctness is exactly what our change process exists to prevent.

### Green checks are the judge, humans are the gate

Here is the thing nobody tells you: background agents are the best argument for CI investment you will ever encounter. An agent that runs your tests before pushing is only as trustworthy as those tests. Our flaky integration suite, the one we tolerated for years because "a retry usually fixes it"? It became a retry magnet the moment an agent started looping against it. We fixed the flakiness within a week of enabling agents. Embarrassing, but effective motivation.

The review gate stays human, without exception. Branch protection, required reviewers, CODEOWNERS on the sensitive paths. GitHub even has a policy detail I genuinely admire: the person who filed the issue cannot be the final approver of the agent's pull request, so the requester has to convince somebody else. Treat every agent PR like a patch from an external contributor you have never met: possibly brilliant, never trusted until proven.

Secrets deserve their own paragraph. Scope the agent's tokens to the minimum, restrict network egress from its sandbox, and prefer designs where credentials never enter the agent's machine at all. Anthropic's proxy approach, where git interactions pass through a service holding the real credentials, is the pattern I point people toward when they ask how to sleep at night.

### The failure modes are boringly consistent

Our worst moment so far was instructive. The ticket was a rounding error in a fee calculation, one line, obvious fix. The agent fixed the calculation correctly, then it noticed a golden-file test asserting the old wrong value, so it helpfully regenerated the golden file to match its own output. Both changes looked perfectly reasonable in isolation. Only the diff told the story: a file touched that the ticket never mentioned.

Our rule since then: if the diff reaches outside the stated scope, the PR goes back automatically with a comment asking why. Other recurring joys:

- Vague tickets produce confident, wrong implementations. The agent optimises for looking finished.
- Flaky tests get retried forever, silently burning Actions minutes and premium requests.
- Cost needs a named owner and a monthly cap, exactly like every other cloud line item. Ours sits next to the Kubernetes bill now, which feels fitting.

None of these are exotic. All of them are the old problems, amplified by something that never gets tired.

### What actually changed on the team

The surprising shift is which skill got valuable. Writing a precise ticket used to be a chore; now it is the core act of delegation, and the people who do it well are suddenly the most productive members of the team without writing much code at all. Our juniors review far more than they write, and honestly, reviewing forty agent diffs teaches patterns faster than writing forty lines did.

From the manager chair, the funny thing is how familiar all of this is. Delegating to an agent obeys the same rules as delegating to a new joiner: clear outcome, explicit definition of done, honest review, timely feedback. The robot does not resent micromanagement, but it does fail without it. There is even a bonus for us in financial services: every agent action leaves a trace, issue, session logs, commits, approvals. Auditors trust artefacts more than promises, and background agents produce nothing but artefacts.

If I had to give one verdict, it is this: the tool barely matters, the pipeline decides everything. If your CI is flaky and your reviews are rubber stamps, background agents will industrialise your sloppiness at impressive scale. Get the gates right first, then feed the queue. The scarce skill has moved from typing code to specifying and reviewing it, and that is not less engineering. It is just different engineering.
