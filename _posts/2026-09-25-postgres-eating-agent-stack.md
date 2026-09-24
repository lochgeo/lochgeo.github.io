---
layout: post
title:  "Postgres Is Eating the Agent Stack"
date:   2026-09-25 05:20:00 +0530
comments: True
categories: [Software, Architecture]
excerpt_separator: "<!--more-->"
---

Two years ago I wrote that in RAG, retrieval is the bug, not the model. I spent that winter fiddling with chunk sizes and embeddings and blaming the ranker every time an answer went sideways. This year I noticed something funny: every agent sketch I draw on a whiteboard ends at the same boring box. Not a shiny new vector startup. Postgres.

A seed note I left myself asked the obvious question: does the AI revolution make the boring relational database more important rather than less? After watching Neon, Supabase, Lakebase, and the Kubernetes folks all converge on the same answer, I think it does.

<!--more-->

### Vectors moved in next door

The first consolidation happened quietly. Instead of running a separate vector database next to Postgres and keeping the two in sync, the embeddings moved into a column.

Supabase's pgvector toolkit is the clearest version of this: store the embedding in a `vector` column, index it with HNSW or IVFFlat, query with cosine or inner-product operators, and — this is the part that sold me — combine it with plain SQL. A similarity search with a `WHERE tenant_id = ...` and a `JOIN` to the orders table is one query, not a distributed transaction across two systems. Their docs list 44 million databases created and 200,000 launching daily, so this is not a lab trick anymore.

Databricks did the same thing for Lakebase this June. Their release notes for June 16 describe Lakebase Search in beta: a `lakebase_vector` extension that speaks the same types and operators as pgvector but uses its own ANN index, plus a `lakebase_text` extension for BM25 keyword search. One database, semantic plus exact-term search, no sidecar search cluster. The indexes are storage-backed, so they survive a scale-to-zero nap without a warmup penalty. I read that detail twice, because anyone who has paid for a search cluster to sit idle over a weekend knows exactly what it costs.

### Branches are the new migrations

The second consolidation is compute separating from storage. Neon, which Databricks acquired in May last year, puts it bluntly on the homepage: separate the two, scale each on its own, let idle databases drop to zero. Their own autoscaling report claims production databases use 2.4x less compute and half the cost versus provisioned — vendor numbers, obviously, but the direction matches what I see.

What changed my mind was branching. Neon and Lakebase both let you branch a database instantly for a pull request or a test without copying the data underneath, because the object storage holds the truth and the compute just points at it. Databricks' own writeup frames Lakebase as compute nodes scaling and branching over shared storage, governed by the same Unity Catalog permissions as everything else.

Neon's about page carries the line that stopped me scrolling: 80% of their databases are now deployed by automated agents. Fifteen million Postgres databases switched on every day, most of them by code I did not write. My tinkerer past — Rancher and k3d on a little IdeaPad, nursing containers through Windows updates — finds this absurd and delightful. My day job in a bank finds it familiar: a branch per change, point-in-time recovery, one permission model. That is the same shape as the eval gate I insisted on last month, just applied to data. I trust a system more when every agent run can point at the exact database snapshot it touched.

### Postgres learned Kubernetes

The third piece is the one I waited longest for. Running stateful Postgres on Kubernetes used to be the thing everyone warned you about at meetups, right after running your own email.

CloudNativePG fixed that by going fully declarative: you describe the cluster — primary plus standbys, replication, credentials, a bit of config — and the operator reconciles it, including failover, with no external failover manager. It joined the CNCF as a Sandbox project in January last year, and 1.30 landed this June. The CNCF page now calls it the most popular Kubernetes operator for PostgreSQL, and for once the marketing matches the hallway track.

I still remember the weekend I stood up Rancher with k3d in Docker on Windows just to learn multi-cluster basics. Back then the database lived safely outside the cluster, because I did not trust anything stateful inside it. These days the operator pattern — immutable images, declared state, self-healing — is how I would run an internal Postgres for agent memory and tool state without thinking twice. EDB even pitched the 1.29 release at KubeCon Europe as a portable, cloud-neutral base for data and AI estates. Strip the sovereign-AI gloss and the point is practical: the same YAML habits my team uses for stateless services now cover the database too.

### My take

So here is my answer to my own seed note. Agents did not kill the relational database. They gave it three new jobs — vector memory, branchable state, and an auditable ledger for everything the agent did — and Postgres absorbed all three because it was already good at the unglamorous parts: transactions, constraints, row-level security, backups.

If I had to pick one database for an agent system in a regulated shop, I would still pick Postgres. Not because it is exciting, but because when the model-risk folks ask what the agent read, wrote, and remembered, I can show them. A vector-only store remembers what things felt like. Postgres remembers what actually happened, and in my line of work that difference is the whole job.
