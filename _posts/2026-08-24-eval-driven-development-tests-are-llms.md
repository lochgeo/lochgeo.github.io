---
layout: post
title:  "Eval-Driven Development - When Your Tests Are LLMs"
date:   2026-08-24 09:45:00 +0530
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

Last quarter, a prompt tweak landed on a Friday. By Monday, 8% of our collateral-margin advisory bot's responses were hallucinating policy clauses that had not existed since the 2021 Basel III update. The change looked harmless in review — just "make the tone more helpful" — but the model took that as permission to improvise. Nobody caught it until a relationship manager flagged a client email. The rollback took twenty minutes. The post-mortem took three days.

That incident is why I no longer ship agent changes without an eval gate. Not "we should really add evals." Not "evals are on the roadmap." The gate is the only way I sleep at night.

<!--more-->

### What Eval-Driven Development Actually Means

Eval-driven development (EDD) is not "write some tests for your prompts." It is the practice of building LLM applications around evals first — before the prompt, before the model choice, before the framework. The eval spec becomes the contract. The dataset is versioned like code. The gate blocks merges that regress.

Braintrust's comparison table nails the shift:

| Dimension | Traditional LLM testing | Eval-driven development |
|-----------|------------------------|-------------------------|
| Test definition | Manual cases written after development | Eval specs defined before changes, updated continuously |
| Pass criteria | Binary pass/fail on exact matches | Scored on multiple quality dimensions with configurable thresholds |
| Dataset management | Static fixtures, rarely updated | Versioned datasets that grow from production traces |
| Failure diagnosis | Manual review of failed outputs | Side-by-side diffs showing which cases regressed and by how much |
| CI/CD integration | Optional or inconsistent | Eval gates block deployments that fail regression thresholds |

The key insight: the eval you build during development is the same eval that gates your releases and monitors production quality. Shared definitions across every stage are what make the system actually work.

### Three Layers, Not One

Agent evaluation is not chat evaluation. A held-out benchmark covers the final answer; production signal comes from the turn. MorphLLM breaks it into three layers that map to how agents actually fail:

1. **Final-answer** — score the last message (correctness, tone, format)
2. **Trajectory** — score the sequence of steps and tool calls (did it call the right API with the right args?)
3. **Per-turn** — score each turn in production (drift, refusal rate, hallucination on turn 3)

Our margin bot failed at layer 2 — the tool call to `get_margin_rule` returned the right data, but the synthesis turn invented a clause. A final-answer eval would have missed it. A trajectory eval caught it.

### The Framework Landscape (August 2026)

The category has settled into five commercial platforms and three open-source standards. Digital Applied's May roundup is still the clearest map:

**Commercial (hosted, SOC 2 matters):**
- **LangSmith** ($39/seat Plus) — LangChain/LangGraph native, lowest SOC 2 entry point, traces land automatically via SDK
- **Braintrust** ($249/mo Pro) — model-agnostic, dataset-first, sandboxed Python custom scorers no one else offers
- **Arize Phoenix** (Enterprise) — OTel-native, agent function-calling eval, self-host option with no event caps
- **Helicone** ($799/mo Team for SOC 2) — observability-first, eval as add-on
- **Promptfoo** (acquired by OpenAI, Mar 2026, $86M) — CLI-driven red-teaming, cross-model matrix testing. The acquisition raises a legitimate objectivity question for non-OpenAI teams.

**Open-source (run your own infra):**
- **DeepEval v4.0.3** — pytest-style, 50+ metrics (G-Eval, task completion, faithfulness), span-level agent scoring, fully local
- **RAGAS** — reference-free RAG metrics (faithfulness, context precision/recall, answer relevancy), Apache 2.0
- **Inspect AI v0.3.225** — UK AISI, native model sandboxing, security eval focus

Most mature teams run two layers: OSS (DeepEval or Promptfoo) for PR-level speed, commercial (LangSmith or Braintrust) for compliance and audit at the production level.

### The Regression Gate That Actually Works

The 2026 practitioner defaults have converged. Genαi's July guide and Kunal Ganglani's three-level framework both point to the same numbers:

- **Sample 5–10% of production traffic asynchronously** for online eval
- **Calibrate LLM judges to 85%+ agreement with human labels** (Zheng et al. 2023 measured position, length, and self-enhancement bias — judges are a cheap proxy, not a final arbiter in high-stakes domains)
- **Block deploys on ~3% pass-rate regression** against the production baseline

A concrete gate config from the Agent Eval Arena project:

```json
{
  "thresholds": {
    "minPassRate": 75,
    "maxRegressionPp": 2,
    "maxCostPerCaseUsd": 0.025
  }
}
```

The decision output is binary: `pass` (promote) or `fail` (block). No "let me check the dashboard." The build fails.

### Cost Economics You Can Explain to Finance

Disciplined teams spend **$8–60 per 1,000 traces** on evaluation all-in. That is a 20–100% increase over an unmonitored baseline, but severe regressions drop by an order of magnitude. The architecture that keeps it sane:

1. **Cheap deterministic checks first** — exact match, schema validation, citation validity, required keyword presence
2. **Classifier fallback** — LlamaGuard 3 1B or Qwen3Guard 0.6B at single-digit ms latency for safety/toxicity
3. **LLM-judge only on cases needing semantic scoring** — the expensive call, reserved for the uncertain 10–20%

Arize's August cost guide confirms: production eval cost extends beyond judge tokens. Trace volume, evaluator coverage, workflow depth, human review, and retention all contribute. Control through sampling, filtering, selective scoring surfaces, and evaluator routing.

In my shop, a custom eval dataset (5K–10K examples reflecting our actual margin rules and client phrasings) cost ~$35K to build and ~$10K/yr to maintain. Integration overhead (CI pipelines, trace storage, dashboarding) added another 20%. The first time the gate caught a prompt regression before it hit a relationship manager, it paid for the year.

### The Regulatory Clock Is Ticking

**EU AI Act high-risk obligations apply from August 2, 2026** — that is not a typo, it is nine days ago. Financial services (credit scoring, fraud detection, insurance underwriting, margin advisory) qualify as high-risk under Article 6. The requirements read like an eval spec: conformity assessments, risk management systems, data governance, technical documentation, **audit logging**, human oversight mechanisms, accuracy and robustness, cybersecurity.

NIST AI RMF and ISO 42001 layer on top. The governance gap is real: most enterprises have agents in production without clear human ownership, audit logging, or defined action boundaries. An eval gate that produces an immutable, correlated trace entry for every tool call — with pass/fail, cost, latency, and judge reasoning attached — is not just engineering hygiene. It is compliance evidence.

### What This Looks Like in Our Pipeline

Our GitHub Actions workflow is boring on purpose:

```yaml
# .github/workflows/agent-evals.yml
on:
  pull_request:
    paths:
      - "prompts/**"
      - "evals/**"
      - "agents/**"

jobs:
  eval-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deterministic layer (Promptfoo)
        run: npx promptfoo@0.105.0 eval -c promptfoo.yml --output results.json
      - name: Semantic layer (DeepEval)
        run: |
          pip install deepeval==2.3.0
          pytest evals/rag/ -v --tb=short
      - name: Regression guard
        run: |
          go build -o guard ./cmd/regression-guard
          ./guard --baseline 0.82 --pr 0.80 --threshold-pp 2
```

The regression guard is a 50-line Go binary that reads the baseline scores from `main`, the PR scores from the run, and exits 1 if any metric drops more than 2 percentage points. It posts a PR comment with the diff. The whole pipeline runs in ~3 minutes.

### The Honest Counterarguments

Heavy eval is over-engineering for low-stakes apps. If a 1% quality drop changes no business outcome, do not instrument for it. And in undisciplined teams, eval cost can hit $2 per $1 of inference while measuring the wrong proxy.

The defense is a **budget per 1,000 traces, set before you build**. Keep the pattern that catches the most failures per dollar and cut the rest. For us, that means deterministic checks on every PR, LLM-judge on 10% of production traces nightly, and a human review queue for the 50 hardest failures each week.

### What It Means for Us

If you are building agents in 2026, "what's your eval architecture?" is as standard a question as "what's your test coverage?" was in 2015. The answer shapes your cost model, your compliance posture, and your user experience in ways that are hard to refactor later.

My pragmatic take: start with DeepEval in your test suite and Promptfoo for prompt regression. Add LangSmith or Braintrust when traffic justifies the SOC 2 ticket. Instrument the eval operations from day one — pass rates, judge agreement, regression events, cost per 1k traces. Treat them like database queries. Because that is exactly what they are.

The prompt tweak that hallucinated a Basel III clause? It never reached a client. The eval gate caught it at 2:47 PM on a Friday. The build failed. The author saw the diff, reverted the "helpful tone" instruction, and the gate passed. Monday morning was boring. That is the goal.