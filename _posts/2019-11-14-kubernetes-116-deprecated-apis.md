---
layout: post
title:  "Kubernetes 1.16 - When the Old APIs Stop Answering"
date:   2019-11-14 20:18:00
categories: [Software, Containers & Kubernetes]
excerpt_separator: "<!--more-->"
---

I have been slowly getting comfortable with Kubernetes after the Docker multi-stage work earlier this year. Mostly YAML, mostly copy-paste from Stack Overflow, mostly "it works on my cluster." Then 1.16 shipped in September and a few of those manifests started getting rejected with errors that looked like the API server had never heard of `extensions/v1beta1`.

That is not a bug. For the first time, Kubernetes actually *stopped serving* a pile of deprecated API versions. If your Deployment still says `apiVersion: extensions/v1beta1`, 1.16 will not politely convert it for you on the way in. It says no.

<!--more-->

### What actually got removed

The official deprecation note is worth reading once with a coffee. The short version of what no longer answers by default in 1.16:

- **Deployments, DaemonSets, ReplicaSets** under `extensions/v1beta1`, `apps/v1beta1`, and `apps/v1beta2` — move to `apps/v1` (stable since 1.9)
- **StatefulSets** under `apps/v1beta1` and `apps/v1beta2` — same destination, `apps/v1`
- **NetworkPolicy** under `extensions/v1beta1` — move to `networking.k8s.io/v1`
- **PodSecurityPolicy** under `extensions/v1beta1` — move to `policy/v1beta1`

Objects already stored in etcd are still there. You can still *read and update* them through the new group/version. What breaks is submitting a create/update that still names the old `apiVersion`. CI pipelines, Helm charts you have not touched in a year, and that one operator that hard-codes client-go types are the usual suspects.

### It is not just a string swap

I made the rookie mistake of a global find-replace from `extensions/v1beta1` to `apps/v1` and assumed I was done. The field defaults changed between those versions. A few that bit me while cleaning a small demo cluster:

- `spec.selector` is required and immutable on Deployment / DaemonSet / StatefulSet / ReplicaSet in `apps/v1`. Older YAML sometimes omitted it and let the controller invent one.
- Deployment `spec.progressDeadlineSeconds` defaults to `600` now (older extensions default was "no deadline").
- `spec.revisionHistoryLimit` defaults to `10` on Deployments (was "keep everything" in some older versions).
- `maxSurge` / `maxUnavailable` default to `25%` instead of `1`.
- DaemonSet and StatefulSet `updateStrategy.type` defaults to `RollingUpdate` instead of `OnDelete`.

None of those are hard. All of them will surprise you in production if you only change the `apiVersion` line and never re-read the object.

### How I actually audited the mess

I am not running a giant multi-tenant platform. I am a .NET person with a few services and a habit of collecting sample manifests. Still, the checklist that worked for me:

1. Grep the repo for the old groups: `extensions/v1beta1`, `apps/v1beta1`, `apps/v1beta2`.
2. For live clusters still on 1.15 or earlier, list what clients are actually *using*: watch API server audit logs or just try `kubectl get` with the old version and see who still talks that way.
3. Convert one object at a time with `kubectl convert` (still around in the 1.16 tooling window) and diff the result before you trust it.
4. Apply the converted YAML to a throwaway namespace on a 1.16 cluster *before* you upgrade anything that matters.
5. Check Helm charts and third-party installers separately. Your app YAML can be clean and the ingress controller chart can still ship beta types.

If you want a brutal dry run on a 1.15 control plane, the release notes show how to start the API server with the deprecated resources disabled via `--runtime-config`. That is a good way to flush out hidden clients without waiting for the real upgrade night.

### CRDs went GA — related, not the same problem

1.16 also graduated CustomResourceDefinitions to `apiextensions.k8s.io/v1`. That is good news if you write operators, and it comes with stricter defaults (structural schemas, pruning unknown fields, and so on). It is a separate migration from the workload API cleanup, but it is the same theme: Kubernetes is done treating "beta forever" as a product strategy. If your CRD YAML still lives on the beta CRD API, put it on the list next to the Deployments.

### What I am doing about it

For me the practical rule is simple. New manifests start on the stable group. Old ones get converted when I touch the service, not "someday." I would rather spend an afternoon on boring `apiVersion` hygiene than explain to someone why a routine cluster upgrade failed because a Deployment written in 2017 still spoke `extensions/v1beta1`.

If you are about to jump to 1.16 — or you already did and something in CI started failing with a version you have never typed by hand — start with the official post [Deprecated APIs Removed In 1.16](https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/) and the [1.16 release announcement](https://kubernetes.io/blog/2019/09/18/kubernetes-1-16-release-announcement/). Then grep your repos. The API server is not going to meet you halfway on this one.
