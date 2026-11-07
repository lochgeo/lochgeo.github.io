---
layout: post
title:  "Manual Override"
date:   2026-11-01 05:20:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

A few weeks ago I watched a video with a title that sounds like a taunt: "AI Was Supposed To Take Your Job. Why Hasn't It?" (Maxinomics, July 2026). Halfway through, it dropped a fact I could not shake off. The US census has been cataloguing occupations for over a century, and for all the noise about machines eating work, history contains only **one** job that a single technology completely erased: the lift operator. Meanwhile the airline pilot — whose routine flying has been largely handed over to computers for decades — is more in demand than ever. Same big wave of automation. Two jobs. Opposite outcomes.

<!--more-->

### The job automation actually killed

The lift operator story is stranger than "the machine replaced the person." Elisha Otis famously demoed his safety brake by chopping the rope on a platform above a crowd — a stunt he staged with P.T. Barnum, of circus fame. But long before the operator left the stage, the *technology to replace him already existed*. A fully automatic push-button elevator was demonstrated at the 1900 Paris Exposition. The economics agreed the whole time: a recent working paper estimates going operator-free saved building owners $2,000–$3,000 per elevator per year in the 1940s (roughly $35,000–50,000 in today's money).

Yet look at the census numbers. In 1900 the US counted about 8,000 operators. By 1950 — with automatic elevators sitting in catalogs for fifty years — there were **85,000**. The job kept growing alongside the machine that would kill it, because passengers genuinely did not trust a steel box nobody was steering, unions negotiated for the human, and building codes in several cities required one. The 1945 New York strike (seventeen thousand operators walked out for five days; office workers climbed thirty flights) finally showed everyone the cost of dependence, and the occupation thinned out fast enough that its last census listing is in 1960. Economist James Bessen's tally: of the 270 detailed occupations in the 1950 census, exactly **one** has been eliminated by automation. This one.

### The job automation "killed" and didn't

Now the pilot. Autopilot dates to the 1920s, fly-by-wire is standard since the 1980s, and autoland can put a plane down in zero visibility. The video interviews a pilot who estimates "I spend most of the flight watching screens and drinking coffee," and closes the loop: "They automated 90% of my job and created ten times more of it."

That is not just feelings; Erik Brynjolfsson has made the same point about jobs and the Jevons paradox. When jets made pilots dramatically more productive, the response was not fewer pilots. Flying got cheap, people flew more than ever, and demand for pilots went up. US Bureau of Labor Statistics data has the whole civil aviation workforce growing by roughly 700,000 jobs — more than 400% — from 1958 to 1996, in the exact decades its core task was being handed to computers.

### Why some jobs amplify and some evaporate

So which way a job tips is not random. Three things decide it:

- **Partial beats total automation.** Bessen again: if a machine takes the *whole* job, employment falls. If it takes *part* of the job, the worker becomes more productive per hour and headcount can rise. Weaving is the textbook case — looms got better, cloth got cheap, factories hired more weavers.
- **The demand has to be elastic.** The Jevons loop for jobs only spins when productivity gains translate into lower prices *and* customers respond by buying a lot more of the thing. We never wanted to fly less just because planes got easier to land. We very rarely want less software. Elastic is our middle name.
- **Non-consumption is the hidden market.** The video borrows Christensen's "competing against non-consumption" idea for its best example: the creator used an AI agent to design, in about half an hour, a CAD part he had wanted for years but never justified spending a whole day learning real CAD software for. That work was never waiting to steal a job from a professional CAD designer — it never existed as a job at all.

For years I had wanted a small compliance exporter to exist — pulling report rows, checking each against a rules list, mailing the summary — and it never made it past a budget committee. Last weekend an agent and I built the whole thing in one sitting: retrieval glue, checks, the lot. Whose job did it take? Nobody's. It competed against nothing.

One Indian footnote: you can still find a liftman in plenty of older apartment buildings here. The paper on elevator operators finds the same thing about 1950s New York — the tech being ready was never enough; habit, union contracts and building codes kept operators in their boxes for years after the push button arrived. Adoption is social, not just technical, which is worth remembering whenever someone hands me a layoff calendar for next quarter.

### Which one is my job

Here is the split that video forced onto my desk. Dev work has a lift-operator half: boilerplate features, CRUD screens, routine tickets, the "summarise this spreadsheet" plumbing. That is the work that was so expensive to produce it only ever got written when a committee approved a budget — and it is exactly the kind of thing agents make nearly free. When production cost collapses and demand is elastic, the automatic elevator arrives. Betting that half of my job is somehow census-proof is exactly the mistake the operators were never in a position to make.

But the pilot half is equally clear where I sit. The production incident at 2 AM. The change I have to explain to the model-risk folks. The PR I put my name on, in a shop where that name has audit weight. Agents can propose; they cannot answer for anything. When everything is on fire, the computer does not go to jail and does not make the call. The video's pilot says it plainly: he is not there to turn knobs, he is there to be "the final human who says this plane is landing safely." That sentence transfers straight to banking.

The uncomfortable part, which I wrote about last month, is that juniors get hired into the lift-operator half first, and it is where all the learning reps used to come from. If the shallow end keeps filling with agents, the human reps have to be manufactured some other way.

### Manual override, on purpose

That is the phrase the title is about. The video's own bet: AI probably won't take your job, it will change it — and "someone using AI will take your job if you do not use AI." I agree with the second half and would sharpen the first: the *label* of my job is far less stable than the *substance* of it. Six in ten jobs that people work today sit in titles that did not exist in 1940, by David Autor's team's reading of the census records — titles like wind turbine technician, cybersecurity analyst, and, per the video's list, dog psychiatrist. The job market never took a poll about its old titles either.

My version of staying the pilot is deliberately boring. Small utilities in Python by hand before I reach for an agent. Read the whole diff, always. When a weird bug shows up, I still get first crack at it without autocomplete. If I let the autopilot have every rep, I will end up as exactly the person trust-in-the-box was supposed to protect — and I already know how the census counts that story.

<iframe width="480" height="320" src="https://www.youtube.com/embed/zRv5kW5mAxM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
