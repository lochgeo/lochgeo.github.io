---
layout: post
title:  "After SolarWinds - Who Builds Your Binaries?"
date:   2021-02-07 11:23:00 +0530
comments: True
categories: [Security]
excerpt_separator: "<!--more-->"
---

For most of my career, "supply chain" meant the warehouse people fretted about. Then SolarWinds happened. A signed Orion update carried a backdoor. Thousands of customers installed it because the signature looked fine and the vendor looked trusted. Suddenly the question was not only "is our code secure?" but "is the *machine that builds our code* secure?"

I spent a weekend walking our .NET services with that question in mind. Spoiler: we had strong opinions about TLS and JWT, and almost no opinions about NuGet restore on the build agent.

<!--more-->

### What actually broke the mental model

The attack was not a clever zero-day in customer firewalls. Attackers got into the *build and release path*, injected malicious code into a DLL, and let the normal signing process bless it. From the outside it looked like a legitimate vendor patch.

That hits different when you live in CI. Every night our Azure DevOps agents pull dozens of NuGet packages, compile a pile of ASP.NET Core APIs, sign nothing of our own in a few places, and push containers. We treat that pipeline like plumbing. After Orion, plumbing is a high-value target.

I keep using a simple analogy with the team: the castle wall is your production network. SolarWinds walked in through the catering truck. The badge at the gate said "approved vendor."

### Map what you actually ship

Before you buy another scanner, open a solution and answer three boring questions:

- Which package sources does restore hit? (`nuget.config` in the repo root, not whatever is on a developer's laptop.)
- Which packages are direct references vs transitive noise?
- Who can push to the feed your CI trusts?

For us the uncomfortable answers were: restore could see nuget.org *and* an internal feed with no clear precedence story, nobody had listed transitive packages in a year, and the service account on the agent had broader feed permissions than any human on the team.

A repo-level `nuget.config` with an explicit `<clear />` on sources is the cheapest hardening step I know. It stops "works on my machine because I have a random source in `%AppData%`" from becoming "works on the agent with a surprise package."

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="contoso-internal" value="https://pkgs.dev.azure.com/contoso/_packaging/internal/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

If you can collapse to a **single private feed that upstreams nuget.org**, even better. One place to audit. One place to revoke. One less "which source won?" mystery during restore.

### Lock the graph you tested

PackageReference is convenient until a floating version or a restored transitive upgrade changes what you ship between the build you tested and the build you released.

We turned on lock files for the services that matter:

```xml
<PropertyGroup>
  <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
  <RestoreLockedMode Condition="'$(ContinuousIntegrationBuild)' == 'true'">true</RestoreLockedMode>
</PropertyGroup>
```

Commit `packages.lock.json`. On CI, locked mode means restore fails if the graph drifts. Locally, developers still update deliberately and check in the new lock file. It is not glamorous. It is the difference between "we know what we built last Tuesday" and "NuGet sorted it out somehow."

Centralizing versions across projects (a single `Directory.Build.props` or Packages.props style list) helps too. When every csproj pins its own Newtonsoft.Json line, audits become archaeology.

### Treat the build agent like production

Most of our threat modeling stopped at the API boundary. Post-SolarWinds, I added a short checklist for the pipeline itself:

- **Least privilege** on the agent identity — feed read for restore, narrow push only where publish needs it, no standing admin on the subscription.
- **No interactive RDP habit** on build machines; if someone needs a box, use a disposable one.
- **Secrets out of logs** — we still catch the occasional `dotnet` command that prints a token when a script goes wrong.
- **Protected branches and required reviews** on the repo that defines the pipeline YAML. Compromising application code is hard; compromising the pipeline definition is often easier and more rewarding.
- **Signed artifacts where we can** — at least know whether the container or nupkg you deploy matches the commit you think it does (digest pins beat floating `:latest`).

None of this requires a new product. It requires admitting the CI box is part of the attack surface.

### Dependencies are code you did not write

Open source is not free of cost; you pay in attention. For a .NET Core 3.1 / .NET 5 estate in early 2021, my practical bar is:

1. Prefer stable, widely used packages over the shiny one from a blog post.
2. Know your top-level references by name — if you cannot explain why a package is there, delete it and see what breaks.
3. Review upgrades that pull new transitive dependencies, not only major version bumps of packages you chose.
4. Watch advisories for the libraries that touch auth, crypto, XML, and serialization — the boring ones that parse untrusted input.

I am not arguing for a freeze. I am arguing against `Install-Package` as a reflex. Every reference is a trust decision.

### What I am not claiming

We did not "solve supply chain security" in a week. Nation-state tradecraft against vendor build systems is above my pay grade. What we *can* do is stop being soft targets on the basics: deterministic restores, explicit sources, locked graphs, tighter agents, and a culture that treats the pipeline as production infrastructure.

If Orion taught the industry one thing, it is that a clean code review on application C# is necessary and not sufficient. Someone still has to ask: who builds the binary, on which machine, from which inputs, and how would we know if any of that changed under our feet?

Start with `nuget.config` and a lock file. Then walk the agent permissions. The catering truck should not get a free pass just because it has a logo on the side.
