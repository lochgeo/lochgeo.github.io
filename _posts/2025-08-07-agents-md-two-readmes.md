---
layout: post
title:  "AGENTS.md - Every Repo Now Needs Two READMEs"
date:   2025-08-07 21:12:00
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

Every coding agent I have picked up this summer has asked me the same question in a different accent. Codex CLI goes looking for an `AGENTS.md`. Gemini CLI wants a `GEMINI.md`. Claude Code reads `CLAUDE.md`. Cursor keeps its own rules folder. Four tools, four filenames, one identical need: *tell me how this repo works before you let me loose in it.*

Last weekend I opened one of my side projects and counted the instruction files piled up in the root. Three of them. All saying roughly the same thing in three dialects. That is when it clicked: my repos need onboarding documents for robots now, the same way they have always needed a README for humans.

<!--more-->

### The same briefing, five dialects

If you have been rotating through agents like I have, you already know the taxonomy:

- **`CLAUDE.md`** — what Claude Code loads into its context when it starts working in your project.
- **`.cursor/rules`** (and the older `.cursorrules`) — Cursor's per-project guidance, with rules you can auto-attach by file glob.
- **`GEMINI.md`** — Gemini CLI arrived in June with this baked in, hierarchical loading included: it walks up from your current directory collecting context files.
- **`AGENTS.md`** — the name Codex CLI settled on back in April, and opencode reads it natively too. Plain Markdown at the repo root, nothing proprietary about it.

Then there is the long tail: `.clinerules` for Cline, `.windsurfrules` over in Windsurf world, Aider's conventions file. Same story everywhere. Every harness reinvented the onboarding doc and gave it its own name.

The cost is not the files themselves, it is the drift. You fix a wrong test command in one file, forget the second copy, and six weeks later an agent is confidently running the stale command. I hit exactly this last month when an agent suggested `dotnet test` inside my Java service, because a global instructions file still carried habits from my old .NET days. The model was not hallucinating. It was obeying an outdated memo I forgot I wrote.

### What actually goes in one

Think of it as the first-week notes you'd hand a new teammate who is keen but has zero context:

```markdown
# Project overview
Spring Boot 3 service for payment reconciliation.

## Build and test
./mvnw verify          # full build + tests
./mvnw -pl core test   # just the core module

## Conventions
- Constructor injection, no field @Autowired
- All money fields are BigDecimal with Currency, never double

## Gotchas
- Local stack needs Testcontainers; Docker must be running
- Never commit anything from src/main/resources/secrets/
```

Notice what belongs there: the boring operational truth that clutters a README if you put it up front. Build commands, style rules, the traps. Notice also what does *not* belong: architecture essays, roadmap dreams, marketing. Keep it short enough that an agent actually reads all of it every run.

### Nested rules beat giant rulebooks

The part I like most is that this converged on hierarchy without anyone coordinating. Codex treats each `AGENTS.md` as governing its directory subtree, with deeper files overriding shallower ones — their own published system prompt spells out that scope rule. Gemini CLI walks up the tree collecting `GEMINI.md` files. So in a monorepo you put repo-wide rules at the root and module-specific rules next to the modules, and the agent picks up the right blend automatically.

That maps to how real teams work. My payments team's rules live near the payments code, not in some central wiki page nobody updates. There is precedent here too: `.editorconfig` quietly solved editor settings the same way years ago — one predictable file, closest one wins, done.

### Rules are prompts, not laws

Now the honest bit, because I keep watching people expect magic. These files are prompts injected into context, not enforced policy. An agent can ignore them, misread them, or run out of attention before it reaches line forty. Three habits make them actually stick:

1. **Keep it lean.** A bloated instruction file gets truncated or skimmed. If everything is important, nothing is.
2. **Pair words with walls.** "Never commit secrets" belongs in the file *and* in a pre-commit hook. Tell the agent the rule; let the linter enforce it. Agents are much better at following rules that CI will catch them breaking anyway.
3. **Treat it as a living document.** When you correct an agent's mistake twice, add a line to the file so the third correction never happens. It is the same muscle as writing postmortem action items.

There is one more benefit that matters a lot in my day job. In a regulated shop, "the model decided" is not an acceptable answer to an auditor. But an instruction file checked into git has a diff, a review, an author, a date. When compliance asks *who told the agent to do X*, the answer is a commit hash, not vibes. That alone makes these files worth having, even if the agents were mediocre.

### The convergence bet

A small movement is already trying to end the filename wars — there is a lightweight spec doing the rounds this summer proposing everyone standardize on `AGENTS.md`, and at least one VS Code agent shipped support for it in late July after an open issue made the interoperability case. My bet is that name wins eventually, simply because it is vendor-neutral. Nobody wants to maintain four copies of the same document forever, and the first tool that reads *everyone else's* filename removes a real tax on multi-tool developers.

Until then: pick your primary, generate the others as thin pointers ("see AGENTS.md"), and move on. Do not wait for the standards process. The pattern itself — versioned, reviewable project briefings written for both humans and machines — is already worth adopting today.

If I had to bet on one habit sticking from this agentic summer, it is this one. Autocomplete changes how you type. Agents change how repos explain themselves. Add the file, keep it honest, and your future teammates — carbon and silicon alike — will thank you.
