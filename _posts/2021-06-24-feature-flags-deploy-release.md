---
layout: post
title:  "Feature Flags - Deploy Dark, Release When Ready"
date:   2021-06-24 20:14:00 +0530
comments: True
categories: [Architecture]
excerpt_separator: "<!--more-->"
---

We got better at shipping code this year. Pipelines build, tests run, images land in the registry, and — if you bought into the GitOps story — the cluster eventually matches Git. That is deploy. What still bites us is treating deploy and *release* as the same button.

I spent the last couple of sprints putting `Microsoft.FeatureManagement` in front of a few half-finished ASP.NET Core endpoints. Not because flags are fashionable. Because product wanted a quiet soak with internal users, and ops wanted a kill switch that did not involve a Friday rollback train.

<!--more-->

### Deploy is not release

Deploy means the binary is running in an environment. Release means a human (or a rule) decided customers should *see* the new behaviour.

When those two are glued together, every merge to main becomes a product decision. Long-lived branches creep back in. Staging becomes a museum of "almost ready" features that cannot go to prod without dragging half-done work along. Someone asks for a one-line config change and you schedule a full release window.

I think of it like loading a truck overnight and opening the shop door in the morning. The truck can sit in the bay. Opening the door is a separate choice.

Pete Hodgson's write-up on [feature toggles](https://martinfowler.com/articles/feature-toggles.html) still has the cleanest vocabulary I have seen. Flags are not one thing. They fall into rough buckets:

- **Release toggles** — hide unfinished work on trunk so you can merge early. Short life. Delete them when the feature is fully on.
- **Experiment toggles** — A/B or canary style: different users, measure, decide. Medium life.
- **Ops toggles** — kill switches and degraded modes under load. May live longer; treat them like runbook controls.
- **Permission toggles** — "only premium" or "only beta tenants." Product configuration dressed as a flag.

If you mix those up, you get permanent `#if` debt and nobody knows who owns the switch.

### What we actually wired in .NET

For ASP.NET Core services we are on the Microsoft stack already, so I started with `Microsoft.FeatureManagement` / `Microsoft.FeatureManagement.AspNetCore`. The library has been around since the 2019 previews and sits on top of the normal `IConfiguration` system. Flags can live in `appsettings`, environment-specific JSON, or a central store later.

Registration is boring on purpose:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddFeatureManagement();
    // ...
}
```

A simple on/off flag in config:

```json
{
  "FeatureManagement": {
    "NewCheckoutFlow": false,
    "BetaReportingApi": true
  }
}
```

In code, inject `IFeatureManager` (or the snapshot variant when you want a consistent answer for one request) and branch:

```csharp
if (await _featureManager.IsEnabledAsync("NewCheckoutFlow"))
{
    return await _newCheckout.ProcessAsync(request);
}

return await _legacyCheckout.ProcessAsync(request);
```

For MVC, `[FeatureGate("BetaReportingApi")]` on a controller or action is enough to 404 the surface when the flag is off. That is nicer than sprinkling the same `if` in every method and forgetting one.

None of this requires a SaaS day one. Start in config. Graduate the *source* of the flags when the team outgrows restarting pods to flip a bit.

### How this pairs with how we deploy

GitOps (or any decent CD loop) answers "is the cluster running the commit we agreed on?" Feature flags answer "is that commit allowed to change user-visible behaviour?"

The shape I want:

1. Merge incomplete work behind a **default-off** release toggle.
2. Pipeline builds and promotes the image as usual.
3. Cluster pulls the new version. Old behaviour still wins for everyone.
4. Flip the flag for internal users, then a percentage, then everyone.
5. Delete the flag and the dead code path. That last step is not optional.

Rollback of a bad *release* is often "turn the flag off" — seconds, no image rebuild, no Git revert circus. Rollback of a bad *deploy* (crash loops, bad schema migration) is still the usual story. Flags do not replace health checks and migrations done carefully. They stop you from using a full redeploy as the only product lever.

### The mess I am already cleaning up

Flags are inventory. Every one is a branch in production clothing.

Things that went wrong on our first pass:

- **No owner, no expiry.** A release toggle from March was still checking a code path nobody could explain. We now put an owner and a "remove by" date in the flag name or a short wiki table. Ugly names beat immortal flags.
- **Testing only the happy path.** CI ran with everything on. Prod ran with most things off. Guess which combination broke. Run critical suites with flags both ways, or at least with the defaults you ship.
- **Flag soup in one method.** Nested `if (flagA && flagB)` is how you invent bugs that only exist for 3% of tenants. Prefer one decision at the edge (controller, handler factory) and keep the domain code dumb.
- **Config only in one environment.** Staging had the new flow on for months; prod never did. The first real traffic found a missing index. Soak with real-ish traffic or accept you are guessing.

I am deliberately *not* buying a full feature-flag platform yet. LaunchDarkly and friends are fine when you need rich targeting, audit trails, and a UI for non-engineers. For a handful of services, `IConfiguration` plus disciplined cleanup is enough to learn the muscle.

### My take

Continuous delivery without feature flags is continuous *deployment of product decisions*. That is a slow, stressful way to ship. Flags let the pipeline stay boring while release stays intentional.

If you only do one thing this month: pick a single in-flight feature, wrap the user-visible entry points with `Microsoft.FeatureManagement`, ship it dark, and practice turning it on for yourselves first. Then delete the flag when it is done. The delete is the proof you are doing flags, not growing a second configuration language you will never finish reading.
