---
layout: post
title:  "Three People, One Problem"
date:   2026-11-13 11:37:45 +0530
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

I have spent most of my career in teams that looked familiar: a product person, a couple of backend engineers, a frontend person, maybe a designer, sometimes an architect floating in and out. Eight to ten people felt normal. Lately, though, I keep hearing a different shape described in calls and articles - three or four people, one of them owning the product direction, shipping what used to take a much bigger crew. My honest view: there is real evidence that some companies are trying this, but it is an early pattern, not a new default.

<!--more-->

### What the evidence actually says

The clearest recent signal I found is from [Bain's 2026 Technology Report](https://www.bain.com/insights/the-half-finished-redesign-how-ai-reshapes-software-organizations-technology-report-2026/). In a survey of 155 software companies, Bain reported that teams have "shrunk to pods of three to five people," with role boundaries blurring as AI takes over more routine execution. Developers spend more time directing agents, and people pick up adjacent work - product managers prototyping, designers writing front-end code, developers handling more testing and deployment.

That sounds close to the "3-4 people including a product person" idea, but it is worth reading carefully. Bain is describing a shift in team shape and responsibilities, not proof that every specialist disappears. It also notes the messy part: in the same survey, 70% of companies still made decisions centrally. Smaller pods do not move faster if approvals and handoffs stay slow.

I also looked at how this shows up inside big companies. In ["A return to two-pizza culture"](https://www.allthingsdistributed.com/2026/06/return-to-two-pizza-culture.html), Werner Vogels describes how the Amazon Quick desktop effort started with a handful of people who prototyped fast, then grew to a small group with a roughly even split between science and engineering before scaling much later as the product took off. That is a strong example of a tiny founding team getting leverage from AI tools. It is not the same thing as saying every mature product team at Amazon is now permanently three people.

Anthropic's research on [how employment is changing there](https://www.anthropic.com/research/employment-at-anthropic) points in a related direction: far more agent work relative to human work, rising output per person, and role shifts across the company. It suggests why smaller teams become possible, but again, it is evidence of leverage and reorganization, not a universal headcount formula.

There are other reported signals too - analysts forecasting "tiny teams" over the next few years, and anecdotes about projects moving from something like eight engineers plus a PM to two engineers plus a PM. I treat those as useful hints, not as proof that the whole industry has flipped.

### Why AI makes small teams plausible

In the old setup, a lot of time went into translating intent across roles and producing first drafts of everything: boilerplate code, test scaffolding, rough UI, glue scripts, docs, even early product experiments. With coding agents and better tooling, one person can cover more of that surface.

When I play with agent harnesses on side projects, I can feel a tiny version of this. I still need to know what I want, catch bad assumptions, and review the result, but the distance from idea to working sketch is much shorter. In a company setting, that compression is what makes a three or four person pod believable: one person close to the customer problem, and two or three builders who can stretch across boundaries with help from tools and platforms.

### What has to change for it to work

This is the part people skip when they get excited about headcount. A small team is fragile if the rest of the system is built for big teams and slow coordination.

- **Clear ownership:** someone has to own the problem end to end, including what "done" means. Without that, a tiny team just becomes a tiny group waiting on everyone else.
- **Decision rights:** Bain's point about central decisions matters. If every meaningful choice needs a distant approval chain, the pod size does not matter much.
- **Strong engineering basics:** good CI/CD, test automation, observability, and a team that can run what it builds. Bain calls out these fundamentals as the base AI-native teams stand on.
- **Loose coupling and platforms:** clear APIs, service boundaries, and self-service platform capabilities reduce how often a small team needs another team in the room.
- **Guardrails, not chaos:** AWS's [two-pizza team](https://aws.amazon.com/executive-insights/content/amazon-two-pizza-team/) write-up makes a practical point - many autonomous teams can create duplication and silos, so governance has to decide when to align or combine efforts.

### The communication question

I had the same doubt: if we create more small teams, do we just create more meetings between teams?

Sometimes, yes. More teams means more boundaries, and boundaries create interfaces. If ten people become three pods with fuzzy ownership and tight dependencies, the Slack threads multiply. Vogels makes this dynamic clear in his description of how growing systems create more coordination and dependencies over time.

But intra-team communication usually drops. Three or four people sitting close to one problem can make local decisions quickly, especially when one of them carries product context. The net effect depends on how the work is sliced. Well-bounded problems, good contracts, and a decent platform can keep most chatter inside the team. Bad boundaries turn "small teams" into a distributed coordination tax.

I don't think the future is a magic number like three or four. I do think AI is making smaller, more blended teams viable in more places than before. If I were trying it, I would start with one messy but bounded problem, give a small group real ownership, and fix the decision bottlenecks before bragging about the org chart.