---
layout: post
title:  "Time to Pray Between Keystrokes"
date:   2026-11-14 08:00:00 +0530
comments: True
categories: [Experiences, Thoughts]
excerpt_separator: "<!--more-->"
---

We went out for dinner last night to take a break from work for a few hours — helped along by a global MS Teams and Outlook outage yesterday that had everyone joking we finally got to go out for dinner thanks to Outlook being down. A few beers in, of course, we ended up talking about engineering culture the entire time anyway. Despite the drinks — or maybe because of them — everyone at the table was sharp, opening up about things that usually get smoothed over in daytime meetings.

At one point the conversation turned to VDIs — the remote virtual desktops many of us had to use while working from India for overseas companies, whether American, European, or Australian. You'd log onto your base laptop, open a VDI hosted in a data centre halfway across the world, and wait. You'd type a line of code and sometimes it would take ten or fifteen seconds for the characters to show up on screen. A friend at the table joked that in that gap you basically had time to pray — pray that the text showed up at all, and hadn't just vanished into the void.

We laughed because all of us knew that exact pause. But what stayed with me wasn't the lag. It was how many smart engineers had treated a ten-second delay between keystrokes as normal. Plenty of folks just lived with it. It was annoying, sure, but not annoying enough to fight over, measure, or escalate. It became the status quo — something you endured rather than something you challenged.

Those VDIs did eventually disappear. Not because management suddenly discovered the latency on their own — the numbers had been obvious to anyone typing through that lag — but because a small, vocal group kept making enough noise that leadership had to admit the setup wasn't acceptable. The friction was identical for everyone; what changed was that a few people refused to treat it as background noise.

I've come to think good engineering culture starts right there: with people who refuse to accept "this is just how it is" when something is clearly broken, slow, or getting in the way of good work.
<!--more-->

### Useful complaints vs. background noise

Not all complaining is equal. Every engineer has sat in rooms drained by the bad kind: vague, repetitive, aimed at people rather than systems, and paired with zero interest in actually fixing anything. I've been around teams where that sort of grumbling became the background music, and nothing ever moved.

The complaint that helps has a completely different shape. It is specific. It ties the irritation to actual cost — this step is slow, this handoff is fragile, this tool is quietly eating an hour a day across ten engineers. And it comes with a next step, even a small one: can we time this, can we test an alternative, can we stop pretending this is fine? That isn't negativity. It's care. It's someone saying the craft matters enough to be dissatisfied with how we're working.

Most of the time, the thing standing in the way is one of the most expensive sentences in software: *"This is how we've always done it."* Occasionally there's a solid reason behind an old rule. Far more often, it's a smell — a sign that a temporary workaround from three years ago calcified into the default after everyone who remembered the original context left.

Pushing back on a calcified default takes some skill. In my experience, pushback lands best when you bring something observable to the table: a benchmark, a log, a reproducible pain, or a rough back-of-the-envelope cost. It also lands better when the person complaining has skin in the game — when you're willing to help clean up the mess rather than just lobbing a grievance over the wall. And it requires psychological safety: people only speak up when they've seen that naming a problem won't get them labelled difficult or disloyal.

Even then, it sometimes backfires. You raise a real issue at the wrong moment, without enough history, or in a room that isn't ready to hear it. That doesn't mean your instinct was wrong; it usually tells you the culture around feedback is brittle, which is useful information in itself. Not every friction is as blatantly obvious as a ten-second VDI lag, which is why teams need people willing to keep pointing at the quieter problems with data in hand.

### Culture is the defaults we stop questioning

If I want to know what a team's engineering culture actually looks like, I don't read the values deck. I watch what people do when the day is busy and something rubs the wrong way. A deploy drags on — does everyone just alt-tab to Teams and wait, or does someone ask why it's slow? A flaky test fails for the third time this week — do we hit rerun and merge anyway, or do we treat a lying test suite as a real bug? A manual script saves an outage once, then twice, and before anyone notices it's the official release process.

I see this constantly with CI pipelines. A build that takes thirty or forty-five minutes starts to feel normal because *"it's running all the static analysis and security scans."* Engineers plan their coffee breaks around it, context-switch onto a second ticket, lose their train of thought, and accept the wait as the price of being thorough. I don't buy that. Thorough checks matter, especially in regulated environments, but a forty-minute feedback loop is still a capability problem. It is always worth asking: what truly needs to run on every commit, what can be cached or parallelised, what belongs in a later stage, and what is this wait actually costing us in broken focus?

Culture is simply the set of defaults a team stops questioning. The dangerous part is how fast our threshold shifts. Ten seconds between keystrokes, or forty minutes for a build, feels outrageous in your first week and completely invisible by month six. That's how mediocrity settles into an engineering org — rarely through one catastrophic bad decision, almost always through a hundred small frictions everyone quietly agrees to tolerate.

### Habits over heroics

Early in my career, I used to admire the engineer who could jump onto a call at midnight, hold a messy system in their head, and pull off a heroic fix while everyone else watched. Over time I've come to value something much quieter. The engineers I most enjoy working with now aren't trying to be heroes. They just have small, considerate habits that keep the rest of us out of midnight calls in the first place.

When something feels slow or confusing, they don't just shrug and work around it — they take five minutes to measure what is actually happening. When a conversation starts going in circles, they'll open Excalidraw or grab a marker and sketch a quick box-and-arrow diagram so nobody feels lost. They jot down the *why* behind a weird decision so the next person reading the code six months later doesn't think they're going crazy. If they stumble over a misleading error message or a broken step in the README, they fix it right then so the person behind them doesn't trip over the same stone.

What makes them easy to work with, though, is that they aren't perfectionists about it. We've all worked under deadlines where you have to ship something a little rough, and nobody enjoys working with someone who turns every code review into a crusade. Good engineers know when "good enough" genuinely is enough for today. They just stay honest about the difference between a conscious shortcut we took with our eyes open, and an old, painful workaround we only keep doing because everyone forgot we were allowed to change it.

### Fresh eyes are an asset, not impatience

Another thread from dinner was about freshers and how teams onboard them. We all know the standard script: a junior engineer joins, spends the first few weeks in quiet listening mode, gets walked through the setup docs, and is subtly expected to absorb the team's habits before having opinions. Context matters, of course — nobody likes the person who wants to rewrite the architecture on day three without understanding why it's built that way — but *"learn how we do things"* very easily curdles into *"don't question how we do things."*

Someone at the table pushed back hard on that onboarding script, and I agreed with them. The newest person on a team has a superpower that expires after about ninety days: they haven't gone blind to the friction yet. They still notice that local setup takes three days of tribal knowledge, or that a deployment step makes no sense.

With coding assistants getting as capable as they are, one friend went further and argued that the gap in raw output between a junior and a senior engineer has shrunk dramatically. I wouldn't go quite that far — experience still shows up in judgment, in debugging when production is on fire, in knowing which failure modes don't show up on the happy path, and in understanding the system beyond the single file open in your editor. Even so, the balance is shifting. And that makes a newcomer's curiosity more valuable, not less. Asking awkward *"why do we do it this way?"* questions isn't impatience; it's one of the highest-leverage contributions a new teammate can make before the status quo starts looking normal to them too.

### Reading every line vs. trusting the guarantees

That brought up my favourite debate of the night. One of my team members said she still wants to read and understand every single line of code that goes into our system. Not out of stubbornness, and not because she refuses to use AI tools, but out of ownership: when something breaks in production, she wants the mental model in her head so she knows where to look and how to fix it. I have a lot of sympathy for that view. Production has a brutal way of punishing fuzzy understanding.

Others at the table challenged her, and their argument wasn't sloppy either. As coding assistants generate more of the implementation, they argued, pretending any of us holds every line by heart doesn't scale. The centre of gravity has to shift toward verifiability: do we have strong guarantees that the system behaves the way we claim? Are the service boundaries crisp, the tests proving real behaviour rather than mocking it away, and the telemetry good enough to tell us when an assumption fails? Can we ship with confidence because the system gives us evidence, rather than because a human memorised every loop?

I don't think you can pick one camp and call it a day. Both instincts are protecting something essential. Wanting to read every line is really about accountability and debugging under pressure; wanting verifiability is about speed, scale, and being honest about the limits of human working memory in a machine-assisted codebase.

The trap is letting either stance decay into a lazy status quo. *"I read every line"* can turn into a slow, ceremonial code review where people nitpick formatting and still miss a broken state transition. *"The tests pass"* can become a convenient excuse for merging a thousand lines nobody on the team could debug at 2 a.m. Good engineering culture keeps the productive tension between the two. It values deep mental models *and* invests heavily in automated guarantees — contract tests, small diffs, observability, clear invariants — while staying uneasy enough to keep asking: *if this breaks tonight, do we actually know why we trust it?*

### Making time to stay sharp

Somewhere around the second round of food, the talk shifted to something less glamorous: simply finding the energy to keep learning.

It is dangerously easy to get trapped in the daily delivery treadmill. You spend all day in tickets, reviews, and calls, come home tired, sleep, wake up, and walk straight back into the same queue. Weeks like that leave zero room to read a long technical post, work through a paper, spin up a weird tool on your laptop over the weekend, or just sit with a new idea until it clicks. Over a year or two, that compounds. Staying current isn't an extracurricular hobby in our field; it's how you keep your judgment calibrated, and how you notice that the way your team builds software has quietly fallen five years behind.

There is an organisational failure mode here too. Plenty of companies try to squeeze the last drop of utilisation out of every developer's calendar, treating any hour not tied to a sprint ticket as slack to be eliminated. On a dashboard, 100% busyness looks like efficiency. In reality, a team that never has breathing room to read, experiment, or step back will faithfully execute today's roadmap using yesterday's thinking. Protecting time to learn — for yourself and for the people around you — is one of the quietest ways of challenging a short-sighted culture.

### When leaders only hear cleaned-up truth

Toward the end of the night we wrestled with a harder question about leadership. We have all worked under or watched engineering leaders who are smart, experienced, and genuinely respected — yet the teams beneath them operate at a frustratingly average level. The software ships, the deadlines are mostly met, nothing is visibly on fire, and yet the output is unmistakably mediocre. Why does that happen under capable leaders?

I don't think there is a single neat culprit, but a few patterns show up repeatedly:

- **Fear of looking negative:** When people sense that bringing up bad news gets them subtly penalized or handed thankless cleanup work, they learn fast what not to say upward.
- **The good-news filter:** Nobody has to lie outright for a leader to go blind. Each layer of reporting rounds off a few rough edges, softens a risk, and frames a painful workaround as "handled," until the picture reaching the top looks tidy and green.
- **Incident-only visibility:** Customer-facing production outages are loud enough to pierce the filter. Almost nothing else is. Internal tools that waste an hour every morning, employee-facing apps that people curse at in private, flaky staging environments, and miserable local dev setups rarely generate a severity-one ticket — so leadership never feels their weight, even while they slowly drain the team's capability and pride.

It made me wonder whether an engineering leader can ever be exceptional in isolation. A leader's judgment and taste only work on the reality they are allowed to see. If the surrounding culture rewards polish and quiet compliance, even a brilliant leader ends up steering by a fictional map.

The same problem runs in reverse when ideas flow from the top down. Good leaders bring ideas — some genuinely sharp, some half-baked, and some flat-out wrong for the system on the ground. Pushing back upward is delicate. A senior leader usually sits across more rooms, sees wider business constraints, and carries experience that deserves real respect. But wider context doesn't make someone right about every technical trade-off.

We didn't settle on a tidy formula at dinner for whose job it is to fix that loop. If a team treats every suggestion from a leader as a mandate, bad ideas get built faithfully and engineers learn to switch off their brains. If people push back defensively on everything without context, trust evaporates. The only workable answer I've seen is shared responsibility: the team owes its leadership a fast, honest, evidence-backed read of the ground truth, and leadership owes the team a room where speaking that truth feels safe and useful.

### What actually helps

The encouraging part of the whole conversation is that you don't need an entire department to wake up at once for culture to shift. A few engineers, consistently refusing to shrug at broken defaults, can raise the floor for everyone around them:

- **Put numbers on the drag.** *"The build is slow"* or *"the VDI lags"* is easy for management to tune out. *"Ten engineers lose forty minutes every afternoon waiting on this step"* turns a private grumble into an engineering problem with a price tag.
- **Fix one sharp edge at a time.** Small, undeniable wins matter. Shaving ten minutes off a pipeline or automating a flaky manual step proves that the pain wasn't a law of nature — it was just an unchallenged habit.
- **Model the useful complaint.** When something is broken, name it specifically, show the impact, and bring a concrete first step. Teammates copy what they see actually working.
- **Raise the floor in the defaults.** Put the better habit into CI checks, templates, test harnesses, diagrams, and runbooks. When the right way becomes the path of least resistance, quality stops depending on who happened to review the PR.
- **Back the person who asks the awkward question.** Especially when it's a junior engineer or a new joiner pointing out that the emperor has no clothes. How a room treats the first honest question sets the rule for everyone else watching.
- **Guard time to think and learn.** Refuse the badge of honour that comes with being 100% booked. A team with no slack to read, tinker, and reflect is already coasting.

None of this flips overnight. Getting rid of those laggy VDIs took months of a small group refusing to shut up and bringing the evidence back to the table every time it got brushed aside. That is how engineering culture actually moves — not through an all-hands slide deck, but through a handful of people who still care enough to get bothered when things are slower, clumsier, or more fragile than they should be.

A team that never complains isn't a happy team. More often, it's a team that has quietly given up believing anything will change. Give me the engineers who still get annoyed by a ten-second pause between keystrokes — and who turn that annoyance into a reason to fix the system.
