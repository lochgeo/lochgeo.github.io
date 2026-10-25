---
layout: post
title:  "Kimi K3.1 Pro - Open Weights Catch Up to the Frontier"
date:   2026-10-25 14:45:00 +0530
comments: True
categories: [Software, Generative AI]
excerpt_separator: <!--more-->
---

Moonshot AI dropped Kimi K3.1 Pro on October 23. It's the first model in their new K3.1 series, a 3T-class open-weight model with a 1-million-token context window. The weights go public by November 4. For anyone tracking the open-vs-proprietary gap, this release matters — it's the closest an open model has come to matching the frontier on coding and agentic benchmarks.

<!--more-->

### What's new

K3.1 Pro is a post-trained version of Kimi K3 (the 2.8T parameter model Moonshot released in July). The difference is post-training compute at scale. Moonshot ran large-scale joint RL across hundreds of downstream domains — coding, 3D modeling, finance, agentic tool use — and optimized every layer of the stack to make each increment of RL compute deliver stable returns.

The result: a model that covers the full range of coding work, from quick fixes to full-project builds across Unity, Unreal Engine, and Godot. It also brings a surprisingly considered visual style to games and web experiences. On the 3D side, it handles parametric modeling for mechanical parts, architectural models, and configurable products — writing and running Blender/CAD scripts without needing MCP connectors.

There's also a Highspeed variant that delivers 2-3× the throughput for more responsive interactions.

### Benchmarks

| Benchmark | Kimi K3.1 Pro | GPT-6 Astra | Claude Opus 5.5 | DeepSeek-V4.1-Flash-1017 |
|-----------|--------------|-------------|-----------------|--------------------------|
| Terminal-Bench 4.0 (coding agents) | **65.7%** | 58.2% | 66.4% | — |
| Kimi Code Bench 3.0 Long-horizon | **61.2%** | — | — | — |
| BenchCAD 3D (with tools, Voxel IoU) | **92.7%** | 95.9% | 96.2% | 81.6% |
| OfficeQA Pro v2 (finance) | **64.4** (SOTA) | — | — | — |
| Hallucination rate (Hard, with tools) | **8.3%** | 5.5% | 15.0% | 23.3% |
| xdev-bench-hard cyber (pass@3) | **81.0%** | 100%* | — | 85.7% |

*GPT-5.6 Sol third-party evaluation

The Terminal-Bench 4.0 number is the headline: 65.7%, ahead of GPT-6 Astra (58.2%) and narrowly behind Claude Opus 5.5 (66.4%). This benchmark evaluates coding agents on realistic, resource-constrained terminal tasks using strict task-specific tests — exactly the kind of work we actually do.

On 3D (BenchCAD), K3.1 Pro scores 92.7% with tools, within striking distance of Opus 5.5 (96.2%) and GPT-6 Astra (95.9%). Without tools it still hits 72.8%, second only to Opus.

The finance numbers are striking for regulated shops: SOTA on OfficeQA Pro v2 (64.4), which tests complex fiscal data analysis over real documents with retrieval, cross-document verification, and multi-step numerical reasoning. Internal benchmarks show 7-11 point gains over K3 on financial modeling and research, and a 4.2× jump on business operation simulations.

Factual accuracy improved dramatically: claim-level hallucination rate on the hard stress test dropped from 23.4% (K3) to 8.3% (K3.1 Pro) with tools. That's better than Opus 5.5 (15.0%) and Fable 5.1 (23.3%) on the same test.

### Pricing

| Model | Input ($/M) | Output ($/M) | Context |
|-------|------------|-------------|---------|
| Kimi K3.1 Pro | 2.00 | 10.00 | 1M |
| Kimi K3.1 Pro Highspeed | 2.00 | 10.00 | 1M |
| Claude Haiku 5.5 | 0.10 | 0.50 | 100K |
| GPT-6 Luna | 0.10 | 0.50 | 272K |
| DeepSeek-V4.1-Flash | 0.15 | 0.60 | — |

At $2/$10 per million tokens, K3.1 Pro sits in flash-tier pricing territory but with a 1M context window — 10× Haiku 5.5's context. Cached input is $0.20/M (per API docs). Zero Data Retention is available for API customers.

### Availability

Live today across the Kimi product family:
- **Kimi.ai** web and mobile apps (iOS, Android, HarmonyOS)
- **Kimi Work** desktop app v3.1.0+ (Windows, Apple Silicon Mac)
- **Kimi Code** via `/model` command
- **Kimi API** at platform.kimi.ai (`kimi-k3.1-pro`, `kimi-k3.1-pro-highspeed`)
- **Kimi Business** for enterprise (off by default at launch)

Full model weights release by **November 4, 2026**.

### What it means

The open-weight frontier just moved. K3.1 Pro isn't "almost there" on coding agents — it's *there*, beating GPT-6 Astra on the most realistic agent benchmark we have and nipping at Opus 5.5's heels. The 1M context and flash-tier pricing make it immediately practical for the long-horizon repo-scale work we actually do.

Moonshot's cyber risk-reduction approach is also notable: they identified and removed cyber-related data from both mid-training and RL stages, dropping xdev-bench-hard performance from 90.5% to 81.0%. That's a deliberate capability ceiling for responsible open release — and the model still improves substantially over K3 (33.3%). The philosophy: diffuse dual-use capabilities gradually rather than concentrating them in a few proprietary systems.

For my shop, the finance numbers are the hook. OfficeQA Pro v2 at 64.4 SOTA means this model can actually handle the multi-document fiscal analysis we throw at agents today. The hallucination rate (8.3% claim-level on hard stress test) is the best I've seen for an open-weight model with tools enabled.

If the weights drop on November 4 as promised, we'll have a frontier-class coding/agent model that runs on our own infrastructure — no data leaving the VPC, no vendor approval chains. That changes the procurement conversation entirely.

The gap isn't closed, but it's no longer a chasm. It's a gap you can plan around.