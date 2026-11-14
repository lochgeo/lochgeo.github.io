---
layout: post
title:  "The Complaint That Helps"
date:   2026-11-14 18:30:00 +0530
comments: True
categories: [Experiences, Thoughts]
excerpt_separator: <!--more-->
---

Over dinner last night, the conversation drifted, as it often does with engineers, into work. Somewhere along the way it veered into engineering culture, and there was a fair bit of complaining at the table. At one point we got talking about VDIs — the remote desktops many of us had to use while working for American companies from India. You'd log onto your base machine, open the VDI, and then wait. You'd type something and sometimes it would take ten or fifteen seconds for the text to even appear on the screen. A friend joked that in that gap you basically had time to pray — pray that it showed up at all, and not that your keystrokes had vanished into the void.

We laughed, because it was painfully familiar. But what stayed with me wasn't the lag. It was how many people had treated that as normal. Plenty of folks just worked with it. The delay was annoying, sure, but it wasn't annoying enough to complain about, push on, or try to change. It became the status quo — something you endured, not something you challenged.

Those VDIs did eventually disappear. Not because the pain was subtle — the data was always there, obvious to anyone who had to work through that lag — but because a small, vocal enough group made enough noise for management to realise this wasn't acceptable. That, to me, is the difference. The friction was the same for everyone; what changed was that some people refused to treat it as background noise.

I've come to think good engineering culture starts right there: with people who refuse to accept "this is just how it is" when something clearly isn't working at its optimal speed or capability.
<!--more-->

### Complaining as care

Not all complaining is equal. There's the kind that drains a room: vague, repetitive, aimed at people, and paired with no interest in change. I've been in teams where that kind of grumbling became the background music, and nothing ever got better.

Then there's the kind that actually moves things. A useful complaint is specific. It's tied to impact — this is slow, this is fragile, this is costing us time or capability. And it usually comes with a next step, even a small one: can we measure it, can we try X, can we stop pretending this is fine? That's not negativity. That's care. It's someone saying the work matters enough to be dissatisfied with how it's going.

The VDI was a good example of this in the small. For a long time the lag was treated as background noise — an annoyance everyone paid individually, in lost seconds, broken flow, and that low-grade frustration of not trusting your own keystrokes. Once a small but vocal group made it a shared problem instead of a private inconvenience, it became a signal that management could act on. The data had been obvious all along; what changed was that someone cared enough to keep pointing at it.

### "This is how we've always done it"

"This is how we've always done it" is one of the most expensive sentences in engineering. Sometimes it's true and harmless. Often it's a smell — a sign that a workaround has calcified into a default and nobody remembers why.

Challenging that safely is a skill. In my experience, pushback lands better when it's grounded in something observable: data, a benchmark, a reproducible pain, a clear cost. It also lands better when the person pushing has some ownership — they're not just throwing a problem over the wall, they're willing to help improve it. Psychological safety matters here too. People speak up when they've seen that naming a problem won't be treated as disloyalty.

And sometimes it backfires. You raise an issue at the wrong time, without enough context, or in a room that isn't ready to hear it. That doesn't mean the instinct was wrong. It usually means the culture around the complaint wasn't ready — which is its own signal. What makes it land, more often than not, is some mix of trust, timing, ownership, and something concrete to point at. The VDI complaints worked because the pain was real, visible, and hard to explain away. Not every friction is that obvious, which is exactly why it needs people willing to keep naming it.

### Culture revealed under load

If I want to understand a team's culture, I don't look at the values on a slide. I look at what happens when things are under pressure. The deploy is slow — does everyone wait, or does someone ask why it's slow? A flaky test fails again — does the team rerun it and move on, or does someone treat it as a bug in the system? A manual workaround saves the day once, then twice, then becomes "the process."

I see the same pattern with CI systems. A build that takes thirty or forty-five minutes starts to feel normal because "it's running all the static analysis and other things." People plan their day around it, context-switch away, and accept the wait as the cost of doing things properly. I don't buy that. Some checks matter a lot, but a slow pipeline is still a capability problem. It's worth challenging: what actually needs to run on every change, what can be parallelised or cached, what belongs later in the pipeline, and what are we paying in lost focus and feedback speed?

Culture is the set of defaults people stop questioning. The scary part is how invisible those defaults become. A ten-second keystroke lag on a VDI, or a forty-minute build, feels outrageous at first and inevitable by the sixth month. That's how mediocrity settles in: not through one big failure, but through a hundred small frictions everyone learns to tolerate.

### Habits over heroics

When I think about what makes a good engineer, I keep coming back to habits rather than heroics. The good engineers I've worked with tend to notice sharp edges early. They measure instead of assuming. They leave systems a little better than they found them. They write things down, draw diagrams, and explain trade-offs so other people can reason about the work too.

They also know when "good enough" really is enough. Challenging the status quo doesn't mean fighting every battle or polishing every corner. It means caring about the things that actually affect speed, capability, reliability, or clarity — and being honest about which compromises are deliberate and which ones are just neglect wearing a familiar face.

### Fresh eyes are not a liability

Another part of yesterday's conversation was about freshers and how teams bring them in. The usual pattern is familiar: a junior engineer joins, spends the first weeks in listening mode, gets told how things work, and is quietly expected to absorb the team's ways before offering opinions. Settling in matters, of course — context matters — but somewhere along the way "learn how we do things" can turn into "don't question how we do things."

That idea got pushed back on at the table, and I liked that. The newest people on a team often see what everyone else has stopped seeing. They haven't yet learned which frictions are "normal," so they ask the awkward questions. With coding assistants getting better, someone even argued that the gap in raw output between a junior and a senior engineer is much smaller than it used to be. I'm not sure I'd state it that strongly — experience still shows up in judgment, in debugging under pressure, in knowing what can go wrong, and in understanding the system beyond the code in front of you. But I do think the balance is shifting.

Which makes curiosity less of a nice-to-have and more of a responsibility. If you're new to a team, unleashing that curiosity — asking why a process is slow, why a workaround exists, why a decision was made — is one of the most valuable things you can do. And if you're part of the existing team, part of good culture is making room for that. The people with the least attachment to the status quo are often the ones best placed to challenge it, as long as we treat their questions as a contribution rather than impatience.

### Knowing every line vs. trusting the guarantees

There was another thread from that same dinner that I've been turning over in my head. One of my team members said she still wants to read and understand every line of code that goes into the system. Not out of stubbornness, but out of responsibility — if something breaks in production, she wants to know the code well enough to understand where to look and how to fix it. That instinct makes a lot of sense to me. Production has a way of punishing fuzzy understanding.

But it was challenged by others at the table, and their point wasn't careless either. In the age of coding assistants, they argued, we're moving away from knowing every line by heart and toward verifiability. The question becomes: do we have good guarantees that the system is working as intended? Are the tests meaningful, the boundaries clear, the behaviour observable? Can I push this with confidence, not because I've memorised every generated line, but because the system gives me evidence?

I don't think this is a clean either/or. Both instincts are reaching for something real. Wanting to understand the code is really about ownership and the ability to debug under pressure. Wanting verifiability is really about scale, speed, and admitting that none of us can hold a large, fast-moving system entirely in our heads — especially when more of the code is machine-assisted. The danger is in taking either side as a comfortable status quo. "I read every line" can become a ritual that misses whether the system actually behaves well. "The tests pass" can become a blanket excuse for not understanding what we just shipped.

Maybe the culture question underneath it is the same one we've been circling: what are we refusing to take on faith? A good engineering culture, to me, makes room for that tension. It values deep understanding *and* it invests in guarantees — code review, tests that prove something, observability, small changes, clear interfaces — so confidence comes from more than hope. The goal isn't to pick a camp. It's to stay uneasy enough to ask: if this breaks tonight, do we actually know why we trust it?

### Making time to stay sharp

We also talked about something less glamorous but just as important: finding time to keep up. It's easy to fall into a cycle where work takes everything — you come home tired, you just want to sleep, you wake up the next morning and jump straight back into the same busy loop. Days like that leave very little room to read a blog, go through a paper, try a new tool, or even sit with an idea long enough to understand it. And over time, that shows. Staying current isn't a side hobby for engineers; it's part of how we grow, how we make better decisions, and how we notice when the status quo around us has quietly fallen behind.

There's an organisational angle here too. Some teams and companies try to squeeze the last drop out of every engineer's day by keeping everyone permanently busy. It can look productive on the surface, but I think it's often counterproductive. If people never get the space to learn, reflect, or explore what's changing in the industry, the team slowly runs on yesterday's thinking. Challenging that kind of busyness is a cultural act in itself — making room, individually and as a team, to stay curious instead of just staying occupied.

### When leaders stop hearing the truth

Another question that came up was about engineering leaders. We've all seen people we consider exceptional leaders, yet the teams under them somehow operate in a not-so-optimal way. The products that come out are fine, maybe even shipped on time, but they're not best in class. They're mediocre. Why does that happen?

I don't think there's a single answer, but a few possibilities are uncomfortable to sit with. One is fear. If people are afraid of reprimand, they learn very quickly what not to say upward. Problems get softened, risks get hidden, and the rough edges never make it into the room where decisions are made. Another is selective reporting — the good news travels up, the bad news gets filtered out. Nobody is lying outright, but the picture the leader sees keeps getting cleaned up on the way.

What that leaves is a broken feedback loop. Often the only signal strong enough to reach the top is something breaking in production. But production is not the whole story. Customer-facing failures are visible, yes, but a lot of engineering friction lives elsewhere: internal tools that waste an hour every day, employee-facing systems nobody complains about loudly, flaky pipelines, painful developer workflows. These things rarely create an incident report, so they rarely bubble up — even though they're exactly where capability and morale leak away.

It makes me wonder whether an engineering leader can really be exceptional in isolation. A leader is only as good as the truth they can hear. Their judgement, taste, and experience matter, of course, but if the organisation around them rewards silence or polish, even a great leader ends up steering with bad information. Part of good engineering culture is building the kind of environment where reporting friction isn't risky, where "this is slow" or "this isn't good enough" can travel upward before it turns into an outage or a mediocre product.

The same tension shows up when ideas flow the other way — from the top down. Leaders push ideas, and some of them are genuinely good, some are mediocre, and some are just bad. Giving quick, honest feedback on those ideas is tricky. A leader is often more experienced, has seen more situations, and talks to a wider set of people than most of us do. That context deserves respect. But "more experienced" doesn't mean "always right." Sometimes an idea doesn't fit the reality on the ground, or misses a constraint the team lives with every day.

We didn't land on a neat answer to whose responsibility it is to push back. If everyone treats the leader's word as final, bad ideas get implemented faithfully and the team learns to switch off its judgement. If people push back carelessly, you create noise and distrust. What I keep coming back to is that good culture has to make that feedback possible in both directions — fast, specific, and grounded in reality — without turning it into either rebellion or blind compliance. The team owes the leader its honest read of the ground truth, and the leader owes the team a place where that read can be spoken.

### How a few good engineers can lift a team

This is the part I keep thinking about: a team doesn't need everyone to be exceptional for its culture to improve. A few people, consistently acting in the right ways, can raise the floor. But it takes deliberate steps.

### What actually helps

- **Make the pain visible.** Put numbers on the drag. "The build feels slow" is easy to ignore; "this adds forty minutes of waiting across the team every day" is harder to shrug off.
- **Fix one sharp edge at a time.** Small, undeniable improvements matter. They prove the situation isn't fixed by nature — it's fixed by habit, and habits can change.
- **Model the useful complaint.** Show what it looks like to raise friction specifically, tie it to impact, and bring a next step. People copy what they see working.
- **Raise the floor, not just the ceiling.** Improve the defaults: CI, tests, docs, review checklists, runbooks. When the better way becomes the easy way, good behaviour stops depending on individual heroism.
- **Protect dissent.** Back the person who points out the awkward thing. If the first person to name a problem gets punished or ignored, everyone else learns the real rule.
- **Protect time to learn.** A team that's always busy but never learning eventually runs on stale assumptions. Making room to read, explore, and think is part of keeping the culture sharp, not a distraction from "real work."

It won't happen overnight. Getting rid of those VDIs took time and a small group willing to keep making the problem visible. But that's often how culture moves — not in a grand transformation, but through a few people who keep asking why a slow, broken, or wasteful thing is still considered acceptable.

A team that never complains isn't a calm team. More often, it's a team that has stopped believing things can get better. The engineers I trust most are the ones who still get bothered — and who turn that bother into something useful.

I should probably add one small confession: we were all a few beers down when this conversation happened, and maybe a little too excited for a night out. But that's often when people really open up. Despite the drinks, everyone was making coherent points, and the discussion just kept flowing. It's funny — we went out to take a break for a few hours and ended up talking about engineering culture the entire time. I suppose when something genuinely bothers you, it finds its way into the conversation one way or another.
