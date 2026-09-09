---
layout: post
title:  "Faster Feelings, Slower Merges - What Our AI Numbers Actually Showed"
date:   2026-05-28 10:15:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

A few weeks ago, someone two levels above me asked a simple question: how much faster is the team with AI coding tools? I opened my mouth, and what came out was a feeling dressed up as a number. Everyone feels faster. I feel faster. The cursor barely rests before the next suggestion lands, and on a good day with a Spring Boot service I have written before, the boilerplate practically types itself.

Then I looked at our actual delivery numbers for the quarter, and the feeling did not survive contact with them. Throughput was flat. Review times were up. That gap between what my fingers tell me and what the merge log says has been living in my head ever since.

<!--more-->

### The study that ruined my slide

I had been halfway through building a deck around our Copilot rollout when I read the METR experiment from last year, and it quietly deleted my favourite slide. METR, a non-profit that evaluates AI systems, ran a randomized trial in early 2025: sixteen experienced open-source developers working in huge repositories they knew intimately, 246 real tasks, each one randomly assigned to allow or forbid AI help, mostly Cursor with Claude 3.5 and 3.7 Sonnet. Beforehand the developers predicted AI would make them 24% faster. Afterwards they still believed it had made them about 20% faster.

The timers said they were 19% slower with AI than without. Not a survey: a randomized trial where the participants misjudged the direction of the effect, not just the size. The researchers' best guesses for why sound painfully familiar to me. The developers accepted less than 44% of what the AI suggested, and the rest of the time went into reviewing, correcting, and re-prompting. On a small unfamiliar script the assistant shines. On a million-line codebase you have carried in your head for years, it mostly offers confident drafts of things you would have written correctly the first time, and charges you review time for the privilege. Add the distraction tax of fiddling with prompts mid-task, and the 19% starts to look less mysterious.

METR themselves stress the limits: early-2025 tools, experts, familiar mega-repos. Their February follow-up with late-2025 agentic tools points the other way, toward real speedups, though with wide uncertainty and a self-selected sample. So the honest reading is not "AI makes you slower." It is that the speedup is not automatic, and your own sense of it cannot be trusted to tell you which side of the line you are on.

### DORA's awkward footnote

The DORA 2024 report tells the same uncomfortable story at organization scale. More than three-quarters of respondents were already relying on AI for at least one daily task, and a 25% rise in AI adoption came with genuinely good things: documentation quality up 7.5%, code quality up 3.4%, review speed up 3.1%. If the report had stopped there, my deck would have survived.

It did not stop there. That same rise in adoption was associated with delivery throughput down 1.5% and delivery stability down 7.2%. DORA's explanation is the one I now repeat to anyone who will listen: when generating code gets cheap, batch sizes get big, and big batches are slower to review and likelier to break things. The bottleneck was never really how fast I can produce lines. It is how fast a second human can understand them, and AI just made my side of that exchange wider while the other side stayed exactly as narrow.

There is a second footnote I like even more. DORA found 39% of developers trust AI output "a little" or "not at all," and trust turns out to be the lever: developers who trust the tool accept more suggestions, search less, and get more of the benefit. Trust is not a personality trait, though. It comes from clear acceptable-use policies and from being given work hours to actually learn the tool instead of poking at it between tickets. In a bank, where every generated line still has to survive the same review and audit gates as a hand-written one, that part hit home. Our governance did not slow the rollout down. It is the only reason anyone trusted it enough to use it properly.

### The churn under the commits

The number that finally killed my deck came from GitClear, which has been measuring what AI-authored code actually looks like across hundreds of millions of lines. Their 2025 update found that moved, refactored code collapsed from about a quarter of all changes in 2021 to under a tenth in 2025, while duplicated code more than doubled, from roughly 8% to 18%. Their earlier 2024 study of 153 million lines showed the same shape forming: more copy-paste, less restructuring.

That matches what I see in our own pull requests. Output is up. Reshaping is down. The assistant is wonderful at adding a new endpoint that looks like the last five endpoints and terrible at telling you the last five endpoints should have been one abstraction. Left alone, a codebase does not rot because people write bad code. It rots because nobody moves code around anymore, and every AI suggestion arrives pre-shaped like the thing next to it. Reviewers, meanwhile, are reading larger diffs of more plausible-looking code, which is the hardest kind of diff to review well.

### What I measure now

I still use the tools daily, and I would not give them up. But I stopped asking "how much faster" and started asking narrower questions that our numbers can actually answer:

- Pull request size: is the average diff growing faster than the feature? A bigger diff for the same story is a cost, not a win.
- Review turnaround: are reviewers taking longer per PR? If yes, the author-side speedup is being paid for on the review side.
- Rework rate: how much of the merged code gets touched again within a few weeks? Churn is where copy-paste goes to confess.
- Throughput of small batches: are we shipping more small changes, or fewer big ones? DORA's small-batch discipline is the whole game now.

None of this is anti-AI. It is the same lesson I learned years ago moving from .NET to Spring Boot: a new tool changes where the time goes before it changes how much time there is. The teams getting real value are the ones that shrank their batches, tightened their tests, and taught people to review generated code with colder eyes than they use on a colleague's. The teams getting theater are the ones counting accepted suggestions and calling it productivity.

So here is my unpopular opinion, earned the hard way: if your AI productivity number comes from how fast it feels, it is wrong in exactly the direction the METR developers got it wrong. Measure the merge, not the keystroke. The feeling is real, and it is also the least reliable instrument on your desk.
