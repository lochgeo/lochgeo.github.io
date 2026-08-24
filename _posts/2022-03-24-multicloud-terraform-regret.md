---
layout: post
title:  "Multi-Cloud - The Slide Deck Promise and the Terraform Hangover"
date:   2022-03-24 10:37:00 +0530
comments: True
categories: [Cloud]
excerpt_separator: "<!--more-->"
---

A few years ago every architecture review had the same slide: three cloud logos, arrows pointing both ways, and the words "avoid vendor lock-in." Nobody wanted to be the person who said "we are an AWS shop" out loud. Multi-cloud sounded like insurance. Cheap insurance, even — Terraform speaks every provider, right?

I have spent enough nights staring at `terraform plan` output that spans two clouds to admit the hangover. The insurance premium was not the invoice. It was the cognitive tax, the state files, and the lie that one HCL dialect makes three platforms feel the same.

<!--more-->

### What we thought we were buying

The pitch was tidy.

- **Exit option.** If one hyperscaler raises prices or has a bad quarter of outages, we move.
- **Best of breed.** BigQuery here, SQS there, AKS because the team already knew it.
- **Compliance theatre.** Some regulator or customer questionnaire asked "multi-cloud strategy?" and a diagram answered yes.

None of that is crazy. Forrester and others have been blunt that single-cloud outages are real and concentration risk is a board topic. HashiCorp's own cloud surveys keep showing multi-cloud as the default operating model on paper. The gap is between "we use more than one cloud" and "the same *workload* can fail over between them without a hero weekend."

Most of what I have seen in the wild is the soft version: SaaS in one place, a data platform in another, primary apps glued to one provider. That is multi-*vendor*, not portable multi-*cloud*. The slide rarely makes that distinction.

### Terraform does not unify the clouds

Terraform is still the best common remote control we have. I mean that. Remote state, plan before apply, modules, CI that posts a diff on the PR — that discipline beats clicking in three portals.

What it does *not* do is give you one mental model. An `aws_instance`, an `azurerm_linux_virtual_machine`, and a `google_compute_instance` are not the same noun with different adjectives. Disk types, identity, networking, images, metadata — different shapes all the way down. A March write-up I bookmarked put it cleanly: the language is shared; the resource schemas are not. You still learn each cloud. You just learn it through provider docs instead of the console.

So the "we hired Terraform people, we are multi-cloud ready" story falls apart the first time someone needs a managed Kafka equivalent on two providers and discovers the knobs do not map. You end up with two modules, two runbooks, and a thin abstraction that everyone is afraid to touch.

### Where the real pain lives: state and blast radius

The bugs that wake me up are rarely syntax. They are state.

- **One fat state file** for "the platform" means a bad apply in networking can hold the lock while app teams wait, or worse, a partial apply leaves half the graph updated.
- **Workspaces for prod vs staging** look elegant until topologies diverge. Prod gets multi-AZ and a WAF; staging does not. Selecting the wrong workspace is a career-limiting autocomplete.
- **Cross-stack `terraform_remote_state`** couples releases. Rename an output in the VPC stack and three app stacks fail plan on Monday morning.
- **`-target` in a panic** "fixes" one resource and quietly desyncs the rest of state from reality. I have done this. I have regretted this.

The patterns that actually help are boring: separate state by change frequency and blast radius (network foundation rarely; app stacks often), remote backends with locking (S3+DynamoDB, Azure Blob, GCS — pick the one that matches the cloud that owns that stack), plan artifacts applied exactly as reviewed, and a hard rule that portal clicks are drift to be burned down, not tribal knowledge.

Multi-cloud multiplies every one of those choices. You do not get three times the resilience. You get three backends, three IAM stories for CI, three ways to lock yourself out of state.

### A more honest strategy

I have stopped arguing for "portable by default." I argue for **intentional placement**.

1. **Pick a primary cloud per product** and go deep. Use that provider's IAM, networking, and observability until the seams hurt for a real reason — not a slide reason.
2. **Use a second cloud for a job, not a mirror.** Analytics, DR backups, a regional constraint, a SaaS that only lives there. Document the *why* in the repo README, not in a deck from 2019.
3. **Keep IaC boring.** Small root modules, versioned shared modules, CI-only apply, drift plan on a schedule. Terraform Cloud or your own pipeline — whatever, just no laptop applies to prod.
4. **Do not abstract too early.** A "cloud agnostic VM module" that takes `cloud = "aws"|"azure"` usually hides the differences until production, which is the worst time to meet them.
5. **Practice the exit you claim.** If lock-in risk is real for a system, a quarterly restore-or-rebuild drill teaches more than a multi-cloud network diagram that has never carried traffic.

### My take

Multi-cloud as a default architecture was oversold. Multi-cloud as a *consequence* of acquisitions, SaaS choices, and a few deliberate bets is just enterprise life. The mistake was pretending Terraform turned that mess into a single skill tree.

I still want everything in Git. I still want a plan I can read on a PR. I am just done paying complexity interest on optionality we never exercise. Pick a home for the workload, fence the state, and spend the saved brain cycles on the product — not on making three clouds pretend they are one.
