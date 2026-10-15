---
layout: post
title:  "The Ladder Lost Its Bottom Rung"
date:   2026-10-15 23:51:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

Two things are true at the same time in my world right now. Every experienced developer I know ships noticeably more with an agent sitting next to them — and every junior I know is finding it harder to get a first job. A note I left myself a while back put it as a question: is the real transition not "fewer programmers" but "fewer programmers per unit of software"? And if so, what happens to the career ladder when the bottom rung disappears?

<!--more-->

### The leverage is real

I will not pretend otherwise. Some mornings I hand a background agent a well-scoped chore, go make coffee, and come back to a pull request that is 80% of the way there. The remaining 20% is mine — the judgment calls, the domain context, the "wait, that will break settlement reporting" bits. But the shape of my week has changed. This matches what Shopify's head of engineering described earlier this year: he can pull up Claude Code or Cursor in fifteen minutes between meetings and build a working prototype, something that used to need a blocked-off day.

The strongest evidence is behavioral, not numerical. When researchers at METR tried to re-run their developer productivity experiment this year, a chunk of developers simply refused to do half their work without AI — even for pay. One of them said doing things the old way felt like walking across the city after getting used to taking an Uber. You do not get that kind of dependence from a tool that does nothing.

### The measurement is messier than the feeling

But here is the part I keep coming back to. That same METR group ran a proper randomized trial last year: sixteen experienced open-source developers, real issues on repos they knew well, coin flip for each task whether AI was allowed. The developers using AI took 19% longer. And here is the kicker — going in, they had predicted AI would make them 24% faster, and coming out, they still believed it had sped them up by 20%. Feeling and stopwatch disagreed completely.

I have a pet theory for why, drawn from my own reviews. The work did not vanish; it moved. Drafting got cheap and reviewing got expensive. I now read diffs like a hawk because I have been burned — a confident-looking change that subtly misunderstands the transaction boundary, the kind of thing I wrote about when I said somebody still has to read the diff. My throughput per week is up, but my throughput per hour of careful attention? Honestly, I am not sure. The Uber is fast, but somebody still has to check the route.

### The rung that is actually missing

That brings me to the juniors, and the part that worries me more than any benchmark. Think about what junior work actually was for most of us: boilerplate, small bugfixes, writing the tests nobody wanted to write, renaming things across a module. Boring? Often. But those were reps. That is how you learn what a transaction boundary is, what a misleading stack trace smells like, what "done" means in a codebase with real users. I learned debugging the way you learn swimming — by being thrown in the shallow end a few hundred times.

If agents now do the shallow-end work, where do the reps come from? A team can absolutely run with fewer programmers per unit of software. What I cannot picture is a team with zero juniors growing its own seniors. Hiring only seniors works until you ask where the next batch of seniors is supposed to come from. It is the classic move of eating your seed corn and calling it efficiency.

In a regulated shop like mine this bites twice. Shopify's engineering head has a line I liked: you can use AI tools, but you still put your name on the PR. Over here, that sentence carries audit weight. Somebody's name goes on the change, somebody explains it to the model risk folks, somebody owns it at 2 AM. A senior can sign that line because she has ten years of reps behind it. A junior who never got the reps cannot — and no agent can sign it for them.

### The counterexample worth copying

So I was glad to find at least one big company refusing to play along. Shopify did not trim its internship program; it grew it from around a hundred interns a year to over a thousand. Their reasoning rang true to me: graduates today grew up with AI the way an earlier generation grew up with phones, and you want those instincts inside the building, asking why things are done the old way.

More useful still is how they interview now, using a framework borrowed from the University of Waterloo. Three modes:

- No AI allowed. Can you write code by hand? Do you know what happens at the layer below the one you work in?
- AI optional. Do you know when to reach for the tool — and when to keep thinking on your own first?
- AI mandatory. The task is too big for the time. Scope it, prompt it, validate it, ship it.

Read that list again. It is not just an interview loop; it is a curriculum. Hand that to a junior and you have basically described what "learning deeply" means in 2026: the fundamentals, the judgment of when to automate, and the craft of driving the machine hard. Their VP admits nobody has figured out evaluation fully yet, which is the most honest sentence in the whole piece.

### My bet

Fewer programmers per unit of software is fine. It might even be great — software that was never worth writing at old prices now gets written. But zero juniors per team is a slow-motion disaster, and it will not show up in any quarterly metric until the seniors start retiring and there is nobody ready behind them.

If I were hiring today, I would steal the Waterloo three modes outright, and I would hire the junior who can answer the first one. The agent can write the code. Somebody still has to understand it, sign the PR, and still be around — and be senior — in five years.
