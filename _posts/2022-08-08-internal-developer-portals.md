---
layout: post
title:  "Internal Developer Portals - Stop Asking Slack Who Owns That Service"
date:   2022-08-08 10:37:00
comments: True
categories: [Software, Architecture]
excerpt_separator: "<!--more-->"
---

Last week I needed the owner of a payment-adjacent API before a release freeze. Three Slack threads, two Confluence pages last edited in 2019, and a Wiki that still pointed at a team that no longer exists. The service was fine. Our *map* of the service was not.

That is the quiet tax of a growing .NET estate: dozens of ASP.NET Core APIs, a few workers, Helm charts in three repos, docs in a fourth place, and ownership living only in someone's head. Platform teams are supposed to fix the infrastructure. Lately I keep hearing a different answer — **internal developer portals** — and Spotify's Backstage is the name that shows up in every architecture review deck this summer.

<!--more-->

### The problem is not Kubernetes. It is discovery.

We already have CI, clusters, and GitOps for the bits that run. What we do not have is a single place that answers boring questions without a human:

- Who owns this service on-call tonight?
- Where is the production README that is not a lie?
- What is the "blessed" way to stand up a new ASP.NET Core API with our pipeline, our probes, and our logging defaults?
- Which teams depend on this library, and will my breaking change light up half the org?

Those questions used to be free when the company fit in one floor. At microservice scale they become a full-time job of asking around. That is not culture. That is missing product.

### Portal vs platform (I keep mixing them up)

People use **IDP** for two different things, and the slide decks do not help.

- An **internal developer platform** is the self-service machinery underneath: account vending, cluster namespaces, golden pipelines, secrets, observability defaults. Think paved road, not a website.
- An **internal developer portal** is the front door: catalog, docs, templates, plugins, the UI where a human starts.

You can have a platform without a pretty portal (YAML + wiki + tribal knowledge). You can also ship a shiny portal that is just a link farm over the same chaos. The useful combination is platform *capabilities* exposed through a portal *product* that developers actually open on Monday morning.

I like the thermostat analogy less here and more of a **reception desk**. The building (platform) still has elevators and power. Reception is where you learn which floor, who to call, and which form not to fill out twice.

### What Backstage actually is in 2022

[Backstage](https://backstage.io/) is Spotify's open-source framework for building developer portals. They open-sourced it in 2020; it entered the CNCF Sandbox that September, and in **March 2022** it moved to **CNCF Incubation** and shipped **Backstage 1.0** — Core, Software Catalog, Software Templates, and TechDocs marked production-ready with a real versioning story. That is why the conversation shifted from "cool Spotify demo" to "can we run this."

The pieces that matter for a day-job .NET shop:

- **Software Catalog** — services, libraries, websites, data jobs, each with an owner, lifecycle, and links. YAML in the repo (`catalog-info.yaml` style) so ownership is code-reviewed like everything else.
- **Software Templates** — cookiecutters with opinions. New service → repo, CI, basic k8s manifests, logging package, docs stub. Ten minutes instead of two days of copy-paste from the last team's half-finished skeleton.
- **TechDocs** — docs-like-code, rendered next to the component. Kill the orphan Confluence page that nobody updates after the rewrite.
- **Plugins** — the extensibility bet. CI status, Kubernetes view for *your* deployments, API specs, scorecards. You grow into it; you do not boil the ocean on day one.

Netflix, American Airlines, Zalando, and a long public adopter list are already on it. That does not mean *you* should clone Spotify's org chart. It means the boring catalog problem is not unique to music streaming.

### Golden paths beat golden cages

The portal only helps if the templates encode a path people *want*. Netflix calls it the paved road; Spotify says golden path. Same idea: default stack that is supported, observed, and secure enough that going off-road is a conscious choice, not the only way to ship.

For us that might look like:

- One ASP.NET Core service template (net6.0, structured logging, health checks, OpenAPI, Dockerfile that is not 800 MB).
- One worker template for queue consumers.
- One library template with packing and a versioning rule people can explain.
- Docs and catalog metadata generated with the repo, not "please remember to register this in the spreadsheet."

If the golden path is slow, ugly, or missing a real requirement (VNet, auth handoff, multi-tenant headers), teams will fork it on day two and you will own two platforms — the official one and the shadow one in every product repo.

### What I would not do on week one

I would not announce "we are a Backstage company" and spend a quarter building plugins nobody asked for.

Order that has worked in conversations with platform folks (and matches what I would push for):

1. **Pick the painful questions.** Ownership and "how do I start a service" beat a beautiful homepage.
2. **Catalog before cosmetics.** Get `catalog-info` into the top twenty services. Empty catalogs train people to ignore the tool.
3. **One template that is honestly better** than copy-paste. Measure time-to-first-PR for a new hire or a new bounded context.
4. **Treat the portal like a product.** Named owners, a backlog fed by app teams, a status story when the catalog is wrong. Platform reliability is the brand; a flaky portal dies faster than a flaky wiki because people expected self-service.
5. **Allow escape hatches.** Aligned autonomy — Spotify's own lesson — beats mandates. If every exception needs a ticket to the platform team, you rebuilt the helpdesk with React.

### The honest risk

Portals fail the same way enterprise service catalogs failed in the SOA years: stale data, mandatory fields nobody believes, and a launch party without a maintenance plan. Backstage does not fix that. Git-backed metadata and docs-in-repo only help if reviews reject PRs that drop ownership or ship without TechDocs the same way they reject broken builds.

The other risk is **portal as theater**. Leadership buys the IDP story, a team stands up the UI, and the actual self-service (namespace, pipeline, secrets, progressive delivery) still runs through three approval chains. Then developers correctly conclude that Slack is faster.

### Closing

I do not need another dashboard. I need fewer archaeology sessions before a change window. A developer portal worth adopting is boring in the best way: accurate ownership, docs next to code, and a template that encodes how *we* ship ASP.NET Core services in 2022 — not how a conference talk ships a demo.

If your org still answers "who owns this?" with @here, start with a catalog and one golden path. Wire Backstage (or whatever portal fits) only as the door in front of that. The door is useless if the building behind it is still a maze — but a building without a door is how you end up in Slack at 5pm asking the same question again.
