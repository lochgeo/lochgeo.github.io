---
layout: post
title:  "GitOps - Stop kubectl-apply-ing Your Way to Prod"
date:   2021-04-16 19:23:00
comments: True
categories: [Software, Containers & Kubernetes]
excerpt_separator: "<!--more-->"
---

Our deploy pipeline used to end with a job that ran `kubectl apply -f` against a folder of rendered YAML. It worked. Until it didn't. Someone would fix a ConfigMap by hand at 11pm, forget to commit it, and three days later the next pipeline run would "helpfully" overwrite the fix. Or the opposite: Git said one thing, the cluster said another, and nobody could tell which was deliberate.

I have been watching the GitOps wave for a while — Flux from Weaveworks, Argo CD from the Intuit folks, now both under the CNCF umbrella. This week Argo CD shipped 2.0. That felt like a good moment to write down what actually changed in how I think about deploys.

<!--more-->

### What GitOps is (without the marketing)

The short version: Git holds the desired state of the cluster. A controller inside the cluster continuously compares that desired state to live state, and either reports the drift or fixes it. Humans stop pushing manifests *into* the cluster from a CI runner. The cluster *pulls*.

That pull model is the part people undersell. Your CI system no longer needs a long-lived kubeconfig with cluster-admin rights. The reconciler already lives in-cluster with the RBAC you gave it. Auditors like "every prod change is a Git commit." On-call likes "the UI shows OutOfSync in red before the pager goes off."

I think of it like a thermostat. You set the temperature in one place. The furnace does not wait for you to walk over and light a match every hour. If someone opens a window, the system notices and corrects — or at least screams.

### Why our old pipeline kept lying to us

A few failure modes I kept hitting with push-style deploys:

- **Drift with no owner.** Hotfix applied with kubectl, never backported. Next release undoes it. Blame goes to "the pipeline."
- **Partial applies.** Job dies mid-manifest list. Half the Deployment rolled, the Service still points at the old selector. Health checks look fine until traffic shifts.
- **Who changed prod?** Cloud audit logs show a service account from CI. Git history shows nothing related. Good luck in the postmortem.
- **Environment snowflakes.** Staging got a manual tweak six months ago. Prod never did. "Works in staging" becomes a joke.

None of these are Kubernetes bugs. They are process bugs that kubectl-from-CI makes easy.

### Argo CD, in practical terms

Argo CD is the tool I have been poking at. It is a declarative GitOps continuous delivery controller for Kubernetes. You point an `Application` CR at a Git repo (plain YAML, Kustomize, or Helm), a path, and a destination cluster/namespace. It syncs. Continuously.

What I actually care about day to day:

- **Sync and health status** on every app — not just "pipeline green," but "cluster matches Git and pods are Ready."
- **Diff view** before you hit Sync. Seeing *exactly* what will change beats reading a 400-line Helm dry-run.
- **SSO and RBAC** so app teams can sync their own apps without a cluster-admin kubeconfig on every laptop.
- **App-of-apps** pattern for bootstrapping: one root Application that points at a folder of other Applications. Useful when you have more than a handful of services.
- **Self-heal and prune** policies when you are ready to trust the loop. Start manual. Turn the automation on after you stop being surprised by the diffs.

Argo CD 2.0 (announced this month on the CNCF blog) adds things that matter once you leave the "one cluster, three apps" demo: a notifications framework (Slack and friends when sync fails), ApplicationSets for multi-cluster / multi-env generation from one template, better UI for large pod sets, and ecosystem pieces like image updater so you are not hand-editing tags in Git every build. The project has been CNCF incubating since 2020; the 2.0 release is less "rewrite" and more "this is how people actually run it at scale."

Flux is the other serious option. More toolkit-shaped, less UI-first, very Kubernetes-native CRDs. Plenty of teams pick it and are happy. I am not doing a bake-off here — pick one, run it for real, stop collecting comparison tabs.

### What I would change in our pipeline tomorrow

I would not throw away CI. CI still builds the image, runs tests, scans the image, and *updates Git* (image tag, Kustomize overlay, Helm value). That commit is the handoff. CD — Argo or Flux — is what makes the cluster catch up.

Concrete shape:

1. App repo builds and pushes `myapi:gitsha`.
2. CI opens a PR (or commits to an env branch) in a **config repo** bumping the tag.
3. Review + merge on the config repo is the promotion gate for that environment.
4. Argo CD sees the commit, syncs, reports health.
5. Rollback is `git revert` (or point the Application at the previous commit), not a panicked `kubectl rollout undo` that leaves Git lying again.

Separate config repo vs monorepo is a taste fight. I like a dedicated config repo for prod because the blast radius of a bad app commit stays in CI, and prod history stays readable. Others keep overlays next to the code. Either works if the *cluster* is not the source of truth.

### The hard parts nobody puts on the slide

Secrets. GitOps does not magically solve them. Sealed Secrets, SOPS, or an external store (Vault, cloud secret manager) with a controller that materializes Secret objects — pick a pattern before you commit your first database password to a private repo "temporarily."

CRDs and order. Applying an Operator and the CRs that need it in one sync can race. Argo sync waves / hooks help; so does not stuffing everything into one giant Application on day one.

Who can merge to main on the config repo. If that branch is wide open, you just moved your prod kubeconfig into GitHub. Branch protection, CODEOWNERS, and required checks are part of the design, not paperwork.

### My take

I am done defending snowflake clusters that only one person can recreate. GitOps is not a religion. It is a thermostat for desired state. Argo CD 2.0 is a solid, UI-friendly way to run that loop on Kubernetes without giving every pipeline a master key.

If your deploys still end in `kubectl apply` from a shared service account, try pointing Argo at *one* non-prod app this month. Live with the OutOfSync badge for a week. The first time it catches a manual tweak you forgot about, you will stop arguing about the philosophy and start arguing about ApplicationSet generators — which is a much better argument to have.
