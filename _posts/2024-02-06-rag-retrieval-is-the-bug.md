---
layout: post
title:  "RAG - When Retrieval Is the Bug, Not the Model"
date:   2024-02-06 11:18:00
comments: True
categories: [Software, Generative AI]
excerpt_separator: "<!--more-->"
---

We spent two sprints blaming GPT-4 for wrong answers in an internal Q&A POC. The model was fine. The chunks were garbage, the top-k was polite noise, and half the "sources" never contained the sentence we needed.

If you are wiring retrieval-augmented generation over real docs in early 2024, the generator is rarely the first place I look anymore. Retrieval quality is.

<!--more-->

### The demo that lied to us

The happy path is seductive. Chunk PDFs, embed them, stuff top-k into the prompt, watch fluent answers with citations. Stakeholders clap. Then someone asks a question that uses the *internal* name for a product, or an error code from a runbook, or a clause number from a policy PDF.

What I saw in practice:

- The right paragraph existed in the corpus.
- Vector search returned three near-miss neighbors that *sounded* related.
- The model answered confidently from the wrong neighbors.
- We filed a "hallucination" ticket against the LLM.

That is not a generation bug. That is a retrieval bug wearing a generation costume.

### Failure modes that keep showing up

A useful checklist showed up in January from an experience report on engineering RAG systems ([Barnett et al., arXiv:2401.05856](https://arxiv.org/abs/2401.05856)). The labels map cleanly to nights I have already had:

- **Missing content** — the answer is not in the corpus; the model should refuse. It often improvises instead.
- **Missed the top ranked** — the answer is in a chunk that never made top-k.
- **Not in context** — retrieved, then dropped by a consolidation or truncation step.
- **Not extracted** — sitting in the prompt, drowned by noise or contradictions.
- **Wrong format / incomplete / wrong specificity** — the model half-follows the instruction even when the evidence is there.

The paper's blunt takeaway matches mine: you do not fully validate a RAG system in a lab notebook. It fails on *unknown* questions in production, and robustness is something you grow, not something you design once.

### Chunking is not a footnote

Everyone underestimates chunking until it hurts.

Fixed-size splits are cheap and honest about their stupidity. They also bisect tables, cut definitions away from the term they define, and glue two unrelated sections because a token counter said 512. Structure-aware splits (headings, sections, element types in financial reports) take more prep and usually retrieve cleaner evidence — there is already early 2024 work on element-based chunking for financial filings that beats naive fixed windows on Q&A tasks.

Rules of thumb I am using right now:

- Prefer boundaries a human would recognize (heading, list item, table) over pure character counts.
- Overlap is a tradeoff, not free insurance. Too much and you pollute ranking with duplicates.
- Chunk size should track *query shape*. Identifier lookups want tight chunks. "Summarize the control framework" wants broader ones. One size is a starting point, not a religion.
- Put metadata on every chunk: source path, section title, page, product, effective date. Filters beat hoping the embedding "understands" tenancy.

### Embeddings just got a real upgrade

On January 25, OpenAI shipped `text-embedding-3-small` and `text-embedding-3-large`, the first major embedding bump since `text-embedding-ada-002` in late 2022. Per their [announcement](https://openai.com/index/new-embedding-models-and-api-updates/):

| Model | MIRACL (avg) | MTEB (avg) | Price / 1k tokens |
| --- | --- | --- | --- |
| text-embedding-ada-002 | 31.4% | 61.0% | $0.0001 |
| text-embedding-3-small | 44.0% | 62.3% | $0.00002 |
| text-embedding-3-large | 54.9% | 64.6% | $0.00013 |

Small is five times cheaper than ada-002 and stronger on multi-language retrieval. Large is the quality pick, up to 3072 dimensions, with a `dimensions` parameter so you can shorten vectors (Matryoshka-style) when your store caps length. If your index is still on ada-002 from last year's spike, a re-embed pass is one of the highest-leverage experiments you can run this month — measure before and after on *your* questions, not only on MTEB screenshots.

### Vectors alone choke on the things ops people type

Semantic search is great at paraphrase. It is mediocre at the tokens that dominate real tickets:

- Error codes (`E0427`)
- API paths (`POST /v2/exports`)
- Clause numbers ("section 7.2")
- Internal codenames that never appear in public training data

BM25-style full-text still wins those. Hybrid retrieval — keyword + vector, merge and re-rank — is the boring architecture that stops looking optional once you leave blog-post corpora. Metadata filters first (tenant, product, doc type), then dual retrieval, then a merge you can explain when audit asks "why that citation?"

### Lost in the middle is real

Even when retrieval works, packing eight chunks into a long prompt is not free. The "lost in the middle" work from 2023 showed models often use the beginning and end of the context better than the middle. Practical mitigations that do not require research code:

- Keep *k* smaller than your ego wants.
- Put the highest-scoring chunk first or last on purpose.
- Ask for citations tied to chunk ids and reject answers that cannot point at evidence.
- Prefer "I don't know from the provided sources" over a smooth guess — especially in a regulated shop.

Longer context windows help capacity. They do not automatically fix ranking or attention bias.

### How I am evaluating this week

I stopped demoing with three cherry-picked questions. The loop now looks like:

1. A fixed set of real questions from Slack and tickets (including ones with no answer in the corpus).
2. Labels for each: which doc/section should be retrieved, what a good refusal looks like.
3. Metrics that separate stages — retrieval hit rate at k, citation accuracy, answer correctness — so I know *which* stage broke.
4. A/B on one change at a time: chunker, embedding model, hybrid weight, prompt. Not all four on a Friday afternoon.

If you only score final answers, you will keep "fixing" the model for sins committed in the indexer.

### Closing

RAG in early 2024 is less "plug ChatGPT into SharePoint" and more classical information retrieval with a fluent renderer on top. The model will sound sure either way. Your job is to make sure the evidence in the prompt earned that confidence.

Upgrade the embeddings if you are still on last year's defaults. Respect keywords. Chunk like a librarian, not like a token counter. And when the answer is wrong, open the retrieved chunks before you open a ticket about GPT-4.

Retrieval is the product. Generation is the UI.
