---
layout: post
title:  "GitHub Copilot - My Pair Who Never Sleeps (And Sometimes Hallucinates)"
date:   2022-01-15 11:18:00
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

I got into the GitHub Copilot technical preview a few months after it shipped, and for a while I treated it like a party trick. Type a method name, Tab through a full body, feel slightly guilty. Then I started using it on real ASP.NET Core work — tests, boilerplate mappers, the dull glue between services — and the guilt shifted. Not "is this cheating," but "am I still reading what I ship?"

We are still in preview as I write this. No paid SKU yet, waitlist energy, grey ghost text in the editor. It is already changing how I start a blank file.

<!--more-->

### What it actually is

GitHub launched Copilot as a technical preview on 29 June 2021, built with OpenAI and powered by [OpenAI Codex](https://openai.com/blog/openai-codex/) — a model trained heavily on public code, not a general chat bot glued onto the IDE. The pitch from the [announcement](https://github.blog/2021-06-29-introducing-github-copilot-ai-pair-programmer/) is simple: it reads the file (and nearby context), and suggests whole lines or entire functions as you type.

It is not IntelliSense with better marketing. IntelliSense knows your symbols and signatures. Copilot guesses *intent* from comments, names, and patterns it has seen a million times on public GitHub. Early on they said it worked especially well for Python, JavaScript, TypeScript, Ruby, and Go. C# is usable in my experience — not magic, but useful — especially when the surrounding code already looks like "normal" .NET style.

I run it mostly in VS Code. There is also a JetBrains plugin and a Neovim path if that is your world. Full Visual Studio integration is still the thing people keep asking for in the hallway; for day-to-day API work I am fine in Code on the WSL side of my laptop.

### Where it earns its keep on a .NET desk

The wins for me are boring, which is the point.

- **Tests first drafts.** I write the arrange block and a comment like `// assert 404 when account missing`, and it often sketches a reasonable xUnit test. I still rewrite asserts and fix fixture names, but I stop staring at an empty `[Fact]`.
- **Mapping and DTO glue.** AutoMapper profiles, manual projections, "take this entity and make that response shape" — it is pattern completion, and patterns are what models are good at.
- **Exploring an unfamiliar API.** New library, thin docs. I type a comment describing the call I want, accept a draft, then *read the actual package docs* before I trust it. Faster than fishing Stack Overflow for the same five lines.
- **Regex and date formats.** Tiny, easy to verify, easy to get wrong by hand. Perfect Tab fodder.

What it is bad at: anything that needs *our* domain. Payment state machines, internal risk codes, the weird legacy flag that means three different things depending on the channel. It will invent a plausible-looking enum value that does not exist. Plausible is dangerous.

I think of it like a very fast junior who has read every public tutorial and none of our Confluence. You would not merge their PR without review. Same rule.

### Habits that keep me honest

After a few weeks I wrote myself three rules on a sticky note:

1. **Never accept a block I have not read.** Grey text is not green tests. Scroll it. Say the logic out loud if it is non-trivial.
2. **Prefer small suggestions.** A full method is fine when the shape is obvious. A multi-file "architecture" dump is how subtle bugs land.
3. **Comments are prompts.** Garbage comment in, garbage code out. A precise one-liner beats a vague `// handle stuff`.

I also turn it off for security-sensitive paths sometimes — auth handlers, crypto wrappers, anything where a confident wrong default is worse than typing slowly. That is taste, not policy. We do not have a company Copilot policy yet; preview tools rarely do.

### The arguments in the office kitchen

Three debates show up every time someone demos it on a call.

**Licensing and training data.** Codex learned from a lot of public code. People worry about license blur and about suggestions that look a little too much like a specific open-source file. Fair worry. I treat every suggestion as if I typed it: I own the license and the bugs. If something looks copied wholesale, I rewrite or drop it. GitHub will have to keep answering the harder legal questions; I am not going to pretend a Tab key settles them.

**Security.** Researchers have already shown models can suggest insecure patterns when the prompt steers that way — classic injection-friendly snippets, weak crypto defaults, that sort of thing. Not unique to Copilot (humans paste the same junk from blogs), but the *speed* means you can accept junk faster. Review and static analysis still matter. Maybe more.

**Skill atrophy.** If juniors only ever Tab-complete, do they learn? Maybe. Same fear we had with Stack Overflow, and before that with IDEs that write properties for you. My take: use it for the mechanical middle, keep doing the hard thinking yourself — design, edge cases, failure modes. If you cannot explain the suggestion, you cannot defend it in a postmortem.

### My take

Copilot will not replace a careful engineer. It will replace some of the minutes we used to spend on boilerplate, and it will occasionally waste minutes on a beautiful wrong answer. Net, for my .NET work, I am faster on the dull parts and I have to stay stricter on review.

I am not turning it off. I am also not letting it drive. The pair who never sleeps is great until you notice they have never been on-call for *your* system — and then you remember why the human still owns the merge button.
