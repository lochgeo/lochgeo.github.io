---
layout: post
title:  "MCP - The Protocol That Might Finally Make Agents Useful"
date:   2025-01-13 09:30:00
comments: True
categories: [Generative AI]
excerpt_separator: "<!--more-->"
---

It is the second week of January 2025. The holiday dust has settled, the first sprint planning of the year is done, and I am staring at a familiar problem: how do I actually get an LLM to *do* something useful with our internal systems without writing a bespoke integration for every single data source?

If you have built anything with LLMs in the past year, you know the drill. You want the model to look up a customer record in Postgres. You write a function. You want it to search Confluence. Another function. Slack? Another function. GitHub? You get the picture. Every new capability means more glue code, more maintenance, more surface area for things to break. In a regulated environment like ours, every integration is also a compliance review, a data-classification exercise, and an audit trail requirement.

<!--more-->

### The standard that arrived in November

Anthropic dropped the Model Context Protocol (MCP) on November 25, 2024. Seven weeks ago. In internet time that is practically ancient history; in enterprise adoption time it is barely a blink. But the signal is interesting: Zed, Replit, Codeium, Sourcegraph, and a handful of others jumped on it immediately. The protocol is open, the SDKs are Python and TypeScript, and the initial server set covers GitHub, Git, Postgres, Slack, Google Drive, and Puppeteer. All local-only for now — remote server support is explicitly "coming soon" in the announcement.

I spent a weekend wiring up the Postgres MCP server against a sanitized copy of our reference data. The experience was... surprisingly smooth. `uvx mcp-server-postgres` (the Python SDK uses `uv` for dependency management, which is its own small delight), point it at a read-only replica, and Claude Desktop can now run `SELECT` queries through a standard JSON-RPC interface. No custom function calling schema. No OpenAPI spec to maintain. Just a protocol that both sides speak.

### What MCP actually solves (and what it doesn't)

The pitch is "LSP for LLM context." Language Server Protocol gave us one editor integration per language instead of one per editor per language. MCP wants to give us one data-source integration per source instead of one per source per AI client. That is a compelling reduction in *combinatorial* complexity.

But the reality on January 13, 2025, is messier:

- **Transport is local only.** The spec defines stdio and HTTP+SSE transports, but the Claude Desktop app only does stdio right now. That means the MCP server runs on your laptop. Fine for a developer inner loop; useless for a shared team agent or a production workflow.
- **Authentication is left as an exercise.** The protocol does not mandate auth. The reference servers mostly rely on environment variables or local config. In a bank, that is a non-starter for anything beyond a sandbox.
- **Discovery is manual.** You configure servers in `claude_desktop_config.json`. There is no registry, no marketplace, no "install this server" button in the UI yet.
- **The client ecosystem is tiny.** Claude Desktop works. Zed works. A few others are experimenting. If your team lives in VS Code or IntelliJ, you are waiting on extensions.

Still, the *direction* is right. The alternative — every vendor building proprietary "agent platforms" with lock-in — is exactly how we got the integration sprawl we are trying to escape.

### The reasoning model context

This is also the moment where "reasoning models" are the conversation. OpenAI's o1 dropped in September 2024. It thinks before it answers. It is expensive (roughly 4x GPT-4o on output tokens) and slow (30-60 second latencies are common). But it *solves* problems that vanilla chat models hallucinate through — multi-step coding tasks, logic puzzles, planning.

The industry assumption in January 2025 is that reasoning will become a commodity capability. Cheaper, faster variants are inevitable. DeepSeek-V3 (December 2024) already showed frontier performance at a fraction of the training cost. The rumor mill says a reasoning variant is imminent. Whether that arrives next week or next quarter, the trajectory is clear: test-time compute is the new scaling law, and the price curve bends down.

What does that have to do with MCP? Everything. A reasoning model that can *plan* a multi-step workflow — "first query the customer DB, then check the transaction log, then format a response" — needs a standard way to *execute* those steps. Function calling works, but it is per-model, per-provider. MCP is the only cross-vendor standard on the table. If o1-class models become affordable enough to run in a loop, the bottleneck shifts from "can the model think?" to "can the model act?" MCP is the answer to the second question.

### Building a server for the day job

Last weekend I wrote a tiny MCP server that wraps our internal feature-flag service. Twenty lines of Python using the official SDK. It exposes two tools: `get_flag` and `list_flags`. The server reads a service-account token from the environment, calls our internal gRPC endpoint (via a sidecar proxy, because of course it does), and returns JSON.

Claude Desktop picks it up. I ask: "Is the new-checkout-flow flag enabled for tenant ACME?" It calls the tool, gets the answer, and replies. The whole loop takes three seconds. Most of that is network latency to the flag service.

The kicker: I can now hand this server to a colleague. They drop it in their config. It works. No Python environment setup beyond `uvx`. No "pip install this, brew install that." The distribution story for *local* MCP servers is genuinely good because `uvx` runs ephemeral environments.

### Where this goes in 2025

I see three threads pulling at once:

1. **Remote MCP servers** — Anthropic has promised enterprise-grade remote server support. When that lands, a team can run a shared Postgres MCP server in their VPC, authenticate via OIDC, and every developer's Claude Desktop (or Zed, or whatever) connects to the same governed endpoint. That is when MCP becomes production infrastructure instead of a desktop toy.
2. **Agent frameworks adopting MCP** — LangChain, LlamaIndex, and the emerging coding-agent harnesses will need a standard tool interface. MCP is the obvious candidate. If the agent loop speaks MCP, you swap the LLM without rewiring your tools.
3. **The registry problem** — Someone will build "npm for MCP servers." Private registries for enterprise, public for open source. Versioning, signing, dependency graphs. It is inevitable and necessary.

### The pragmatic take

Do not rewrite your integration layer for MCP tomorrow. The spec is v0.1. The transport story is incomplete. The auth story is absent. The client support is narrow.

But — *do* build your next internal tool as an MCP server. It costs you an afternoon. It forces you to define a clean, schema-validated interface (the SDK makes you). It gives you a local development path that *feels* like the future. And when remote support lands, you are one config change away from a shared, auditable, compliant agent gateway.

We have spent years building APIs for humans (REST, GraphQL, gRPC). MCP is the first serious attempt at an API *for models*. The ergonomics are different — tools not endpoints, schemas not Swagger, stdio not HTTPS — but the principle is the same: standardize the contract, decouple the clients, own the implementation.

I am keeping the feature-flag server. I am adding a read-only Postgres server for the analytics schema next. And I am watching the spec repo like a hawk.

Because if 2025 is the year agents actually ship in the enterprise, MCP — or something very like it — is the plumbing they will run on. And for once, the plumbing might be standardized *before* we bury it in concrete.

{% include pullquote.html quote="Standardize the contract, decouple the clients, own the implementation." %}