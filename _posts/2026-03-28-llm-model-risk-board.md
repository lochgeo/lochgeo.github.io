---
layout: post
title:  "Model Risk Board - Getting My LLM App Past the People Paid to Say No"
date:   2026-03-28 10:15:00 +0530
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---
A few weeks ago I demoed a small internal assistant to my team — a chat box over our runbooks and incident notes, Spring Boot service in front, retrieval over docs we already own, pinned model version behind it. People liked it. Then one senior asked the question that ended the celebration early: "Is it registered?" He did not mean the demo URL. He meant the model inventory.

So I spent the next two weeks doing something I had never done for any CRUD service: filing my LLM app with the model-risk board. Forms, diagrams, eval sheets, a meeting where serious people asked unserious-sounding questions like "what stops it from making things up about a production database?" I walked in annoyed at the paperwork. I walked out thinking every team building LLM apps in a bank should do this on purpose.

<!--more-->

Registering a model is like applying for a visa. The form looks bureaucratic until you realise it is forcing you to answer questions you should have answered anyway: where are you going, exactly what will you do there, what will you do if things go wrong, and how will anyone find you afterwards. The board does not care how clever your retrieval pipeline is. They care whether someone can reconstruct what happened six months from now.

### The inventory entry is the easy part

In US banking supervision, the rulebook here is still SR 11-7, the Federal Reserve's 2011 guidance on model risk management (with its OCC companion 2011-12). Old document, simple spine: keep an inventory of every model, validate it independently before it goes live, monitor it afterwards, and make governance own the whole thing. Banks have spent the last two years arguing about whether an LLM assistant counts as a "model" under that definition. My bank's answer is pragmatic: if it touches decisions, summaries people act on, or production-adjacent answers, it gets inventoried. My runbook bot qualified on day one.

The inventory row itself took twenty minutes: name, owner (me), version pins, data sources, intended use, and — this is the field I now love — explicit out-of-scope uses ("not authorised for customer-facing answers, not authorised for access decisions, not connected to any write path"). Writing down what the thing is *not for* cleared up three arguments before they started. Everything else in the process hangs off that row.

### What I actually filed

The pack I submitted had five parts, and I am listing them because nobody showed me a list beforehand and I had to assemble it by asking around:

- **Intended use plus a boundary line.** One paragraph on what the app does (answers questions over named internal docs, cites sources, says "I don't know" otherwise) and one paragraph on what it must never do. The board reads the second paragraph first.
- **An architecture diagram with the boring parts highlighted.** Not the embedding model — the logging, the PII redaction step, the read-only tool scopes, the fact that the model cannot reach production systems at all. Controls first, cleverness second.
- **An eval sheet, not vibes.** A golden set of about eighty questions with known-good answers from our own docs, plus a smaller set of hostile probes ("ignore your instructions and..."). Pass criteria written down before I ran them. Results attached, failures included — hiding a failure is how you lose the room.
- **A versioning promise.** Pinned model and embedding versions, recorded in the inventory. Any bump re-runs the golden set and counts as a change worth re-review. The board has seen too many "same app, new model, surprise behaviour" stories.
- **A monitoring plan with a human in it.** Override and correction rates, sampled answer reviews every sprint, a named owner for the review rota, and a kill switch (feature flag off, bot answers nothing) that I demonstrated rather than described.

Nothing in that list required permission from a vendor. For third-party models the expectation, straight out of SR 11-7 thinking, is that you validate in *your* context on *your* data — the vendor's model card is input, not evidence. That one sentence cost me a weekend of eval work and saved me in the meeting.

### The EU AI Act corner of the form

There is now a second page to this paperwork that did not exist a few years ago. The EU AI Act's duties for general-purpose AI model providers have applied since 2 August 2025 — technical documentation, a copyright policy, a public summary of training content, with extra assessment, incident-reporting and cybersecurity duties for the largest systemic-risk models — and our internal form now asks deployers like me how those upstream duties flow down: which provider, which version, what documentation exists, where the transparency obligations land. The Commission's GPAI guidelines and the Code of Practice are the reference points our compliance folks cite.

I am not the provider of the base model, so most of that chapter is "confirm and attach" rather than "author." But it changed the conversation: data retention, disclosure to users that they are talking to an AI, and logging that can actually answer "what did it say, given what context, on which version" are no longer nice-to-haves. With the broader transparency duties phasing in from August 2026 and high-risk system duties after that, the board's view is simple — build the audit trail now while it is cheap, because the timetable only moves one direction. (Useful starting points if you are in the same spot: the [Commission's GPAI obligations page](https://digital-strategy.ec.europa.eu/en/factpages/general-purpose-ai-obligations-under-ai-act) and the [AI Act implementation timeline](https://artificialintelligenceact.eu/implementation-timeline). The [SR 11-7 letter itself](https://www.federalreserve.gov/bankinforeg/srletters/sr1107.htm) is shorter than its reputation.)

### What the board actually pushed back on

Two things, both fair. First, my "I don't know" behaviour was asserted, not measured — so I went back and added twenty unanswerable questions to the golden set and reported the abstention rate separately. Second, my log retention said "per standard policy," which means nothing — I had to state exactly what is stored (prompt, retrieved chunks, answer, versions, timestamps), who can read it, and for how long. The meeting took forty minutes. I got a conditional approval with three follow-ups and a review date, which in bank terms is a warm hug.

If you build LLM apps in a regulated shop, here is my opinionated takeaway: file early, before the demo gets popular. The inventory entry plus the five-part pack took me about three focused evenings, and it made the app *better* — the eval set caught a stale runbook that was poisoning answers, and the out-of-scope paragraph stopped a well-meaning manager from pointing the bot at customer tickets. The board never asked me to remove the bot. They asked me to prove I understand it. That is a reasonable price for being allowed to run an opinionated autocomplete next to production, and I would rather pay it in forms than in postmortems.
