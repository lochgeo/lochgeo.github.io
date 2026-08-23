---
layout: post
title:  "Remote Pairing - When the Desk Next Door Becomes a Link"
date:   2020-04-16 11:18:00
categories: [Personal Development]
excerpt_separator: "<!--more-->"
---

Three weeks into working from home full time, I realised the thing I missed most was not the office coffee. It was the five-minute shoulder tap. "Can you look at this exception with me?" used to mean rolling a chair over. Now it means scheduling a call, fighting the VPN, and hoping the screen share does not freeze mid-stack-trace.

I am a .NET person. Most of my day lives in Visual Studio on Windows. So when the whole team got shoved onto home broadband overnight, I needed a pairing setup that did not feel like watching someone else drive through a foggy window. Visual Studio Live Share turned out to be the least bad answer we have found so far — and on a few days, genuinely better than the old way.

<!--more-->

### What broke when we left the office

The technical work did not suddenly get harder. The *coordination* did.

- You cannot overhear that two people are stuck on the same flaky integration test.
- Whiteboard design turns into "hold on, let me open Paint" or a camera pointed at a notebook nobody can read.
- Junior folks stop asking small questions because a Teams call feels heavier than leaning over a monitor.
- Debugging a local-only repro becomes a story you tell, not a state you share.

We still had Slack and Teams. Those keep chat and standups alive. They do almost nothing for "both of us, same solution, same breakpoint, right now."

### Screen share is not pairing

My first instinct was the obvious one: share my whole desktop on Teams and talk. It works for a demo. It is miserable for real pairing.

Only one person has a usable keyboard. The other is a spectator with a laggy picture of your IDE theme. Switching driver means stopping the share, starting theirs, losing the open files and the debug session. On a mediocre home connection the video of a 4K IDE is the first thing that chokes — and you still cannot click their editor.

That is presentation, not collaboration.

### Live Share: share the session, not the pixels

Live Share has been shipping with Visual Studio 2019 for about a year now, and there is a VS Code extension too. The model is different from screen share: you invite someone into *your editing and debug context*. They get their own IDE view of the same workspace. Cursors, edits, breakpoints, and the debug session can be shared without pushing a video stream of your monitor.

In practice, for us, a session looks like this:

1. I hit Live Share in VS, copy the link into the team channel (or DM).
2. They join from Visual Studio or VS Code. Lately guests can even join from a browser if they do not have the IDE ready — useful when someone is on a locked-down machine.
3. We pick a driver/navigator rhythm. One person types; the other watches the diff, suggests the next step, and keeps the bigger picture.
4. When we need to swap, they just start editing. No "can you give me control?" dance.

The bit that sold me was joint debugging. Host hits F5, guest sees the same breakpoint hit, same locals, same call stack. For a dirty write or a race that only shows up under a particular request path, that is gold. Explaining it over chat was always lossy. Stepping through it together is not.

Microsoft has been writing about exactly this remote-dev use case the last couple of weeks, which matches what we are stumbling into on our own. The tools existed; the urgency did not, until March.

### What still hurts

Live Share is not magic, and lockdown does not forgive weak plumbing.

- **VPN first.** If the repo, NuGet feed, or internal API only exists behind the corporate VPN, both of you need a stable tunnel before the session starts. Half our "pairing is broken" tickets were really "someone's VPN dropped."
- **Bandwidth.** Live Share is lighter than full desktop video, but audio still matters. When the home Wi-Fi is garbage, we put the call on the phone and keep the IDE session on the laptop. Ugly. Effective.
- **Permissions and secrets.** Do not casually share a workspace that has production connection strings in `appsettings.Development.json`. Treat a Live Share invite like giving someone a seat at your machine — because you basically are.
- **Focus.** At home there is always another tab, a delivery at the door, a family member on another call. Pairing only works if both people close the noise for that block of time. We time-box sessions to 45–60 minutes and take a real break.

Also: not every task needs two people. Spike work, reading a design doc, or grinding through a pile of similar unit tests is often faster solo. We mark pairing candidates in grooming now instead of defaulting everything to a shared session.

### Habits that actually helped

A few boring process tweaks mattered more than any one tool:

- **Default to video on for the first five minutes**, then mute video if the connection is tight. Faces still help when you are onboarding someone; after that, audio + IDE is enough.
- **Say what you are about to do.** "I am going to extract this method" beats silent typing while your pair wonders if you froze.
- **Write the decision down.** The hallway agreement is gone. A three-line note in the PR or the ticket is the new hallway.
- **Keep a shared "how we remote" page** — VPN quirks, which feed to use, Live Share vs plain screen share, who owns the build agents. Ours started as a messy OneNote and is already the most linked doc on the team.

None of this is sophisticated. It is just making the invisible office glue explicit.

### Will I go back to only shoulder-taps?

When we eventually return to a building, I still want the option to spin up a Live Share session with someone in another city — or someone at home with a sick kid. Distributed collaboration stopped being a nice-to-have the week the offices closed. The teams that already treated chat, tickets, and shared editors as first-class are coping. The ones that ran on proximity alone are rediscovering why documentation and pair habits matter.

If you are a Visual Studio shop and you have not tried Live Share under real remote load yet, spend one afternoon on a real bug with a teammate. Not a hello-world demo — a nasty one. That is the test. For me, the desk next door is a link now, and some days that is enough to keep the work moving.
