---
layout: post
title:  "Agent Memory - When Your Assistant Remembers Too Much (Or Too Little)"
date:   2026-07-28 09:30:00
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

The other day, a colleague asked our internal coding agent to "refactor the fee calculation module like we discussed last month." The agent cheerfully rewrote the whole thing — except we'd never discussed it. It had hallucinated a conversation from thin air, stitched together fragments from three unrelated PR reviews, and produced code that compiled but violated a regulatory constraint we'd explicitly documented six months ago.

That incident crystallized something I've been wrestling with: memory in agent systems is not a solved problem. It's the new capacity planning.

<!--more-->

### Why Context Windows Aren't Enough

You'd think a 1-million-token context window would make memory obsolete. It doesn't, for three reasons that show up in production before they show up in benchmarks.

First, **context rot**. Stuff 800K tokens into a prompt and the model starts losing the middle — the "lost in the middle" effect is real and measurable. I've watched retrieval quality degrade noticeably past the 200K mark on our advisory chatbot, and the academic literature backs it up.

Second, **token economics**. At frontier API prices, a 600K-token context per turn is real infrastructure money. The LoCoMo benchmark shows full-context approaches burning ~26K tokens per query versus ~1.8K with selective memory — a 90% cost cut for roughly 6 accuracy points of trade-off. In a regulated shop where every API call needs an audit trail, that difference compounds fast.

Third, **cross-session learning**. Agents need to remember your preference for Svelte over React, the fact that the collateral margin rule changed in Q3, and that the last three attempts to parallelize the batch job all failed for the same reason. Context windows reset every conversation. Memory systems don't.

### The OS-Style Tiered Model

The dominant architecture by mid-2026 comes straight from the MemGPT paper: treat the agent's memory like an operating system treats RAM and disk.

| Tier | Purpose | Analogy | Access Pattern |
|------|---------|---------|----------------|
| **Core memory** | Always in-context: user persona, agent persona, current task | CPU registers | Zero-latency, always present |
| **Recall memory** | Conversation history, scrollable, searchable | RAM | Explicit `search()` / `swap()` tool calls |
| **Archival memory** | External vector store: massive, cheap, retrieved on demand | Disk | Explicit `insert()` / `search()` tool calls |

The key insight: the **agent decides what moves between tiers**. It's not a background process — the model calls `memory.insert()` when it learns something new, `memory.search()` when it needs a fact, `memory.swap()` when recall gets too full. This explicit control is what separates the MemGPT/Letta pattern from naive RAG.

The numbers bear it out. MemGPT hit 93.4% on the Deep Memory Retrieval benchmark. Zep's temporal knowledge graph edged it at 94.8% (GPT-4 Turbo) and 98.2% (GPT-4o Mini). On LoCoMo, Mem0's 2026 algorithm scored 92.5 overall at ~6,800 tokens per retrieval versus 25,000+ for full context.

### The Framework Landscape (July 2026)

Four frameworks dominate the conversation, each with a distinct memory philosophy:

| Framework | Memory Model | Best For | Hosting |
|-----------|--------------|----------|---------|
| **LangMem** | Episodic + semantic + procedural; background extraction | LangGraph teams | Self-host |
| **Letta (ex-MemGPT)** | Core + recall + archival; explicit tier tools | Long-running agents needing OS-style control | Self-host / cloud |
| **Mem0** | Semantic facts with auto-extraction | Fastest managed integration | Managed service |
| **Zep** | Temporal knowledge graph (bi-temporal: event + system timeline) | Time-aware queries, compliance | Cloud / self-host |

LangMem's background extraction is convenient but adds latency — p50 ~18s, p95 ~60s per the vectorize.io benchmarks. That's fine for async workflows, painful for interactive agents. Mem0's managed service is the quickest to drop in; their BEAM benchmark shows 64.1% at 1M tokens and 48.6% at 10M tokens, which is honestly where most of us live. Zep's temporal KG is the only one that natively models "I used React last month, we migrated to Svelte this month" as a first-class transition rather than a conflict to resolve.

### The Production Gotchas Nobody Talks About

**Forgetting policies.** Not everything should be remembered forever. We need explicit eviction — TTL on archival facts, decay on episodic recall, and a "right to be forgotten" mechanism for compliance. I've seen memory stores grow until the vector search latency itself becomes the bottleneck.

**Memory reconciliation.** The React-to-Svelte example isn't theoretical. In finance, a client's risk tolerance might shift from "aggressive" to "conservative" after a market event. The memory system needs to recognize this as an update, not an append. Most frameworks leave this to you.

**Auditability.** My day job requires provenance: who/what inserted this fact, when, and under what policy. Mem0 and Zep have decent audit trails; LangMem's background extraction makes this harder because the "who" is a background job, not the user or the agent turn.

**Latency vs. quality trade-off.** The LoCoMo numbers are stark: full context gives 72.9% accuracy at 9.87s median latency; Mem0 selective gives 66.9% at 0.71s. That's a 91% latency cut for ~6 accuracy points. For a high-frequency trading advisory bot, I'd take the speed. For a regulatory filing assistant, I'd take the accuracy and optimize the prompt instead.

### What This Means for Us

If you're building agents in 2026, "what's your memory architecture?" is as standard a question as "what's your database?" was in 2015. The answer shapes your cost model, your compliance posture, and your user experience in ways that are hard to refactor later.

My pragmatic take: start with Mem0 if you need something working this sprint and can tolerate a managed dependency. Graduate to Letta when you need explicit tier control and can invest in self-hosting. Reach for Zep when temporal queries or audit trails are first-class requirements. LangMem fits if you're already deep in LangGraph and can absorb the extraction latency.

And whatever you choose, instrument the memory operations from day one. Token counts, retrieval latency, hit rates, eviction events — treat them like database queries. Because that's exactly what they are.

The agent that remembered a conversation we never had? We added a simple verification step: before acting on "remembered" context, the agent now cites its source span. Hallucinated memories don't have spans. Real ones do. Sometimes the best memory system is the one that knows when to say "I don't recall."