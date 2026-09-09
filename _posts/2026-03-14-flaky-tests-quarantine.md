---
layout: post
title:  "Flaky Tests - Stop Hitting Rerun and Start Quarantining on Purpose"
date:   2026-03-14 10:30:00 +0530
comments: True
categories: [Software, Debugging]
excerpt_separator: "<!--more-->"
---
Our Spring Boot services build on GitHub Actions ten, fifteen times a day, and for a month the pipeline had a habit I hated: a different integration test failed every other run, somebody hit re-run failed jobs, it went green, everybody moved on. The suite was green on paper and nobody believed a word it said.

Last month it bit us properly. An `OrderStatusIT` test failed on my PR, passed on rerun, failed again on a teammate's completely unrelated PR, and blocked three merges in one afternoon. The failure had nothing to do with any of our changes. That evening I stopped asking "whose change broke this" and started asking the better question: why does one unreliable test get to hold the whole team hostage?

<!--more-->

A flaky test is like a smoke alarm with a dying battery. It chirps at 3 AM for no reason, so you learn to sleep through it — and then one day there is a real fire and you do not even roll over. Every blind rerun trains the team the same way: red means nothing, green means "try again until it passes." That is how you end up merging through noise.

### The rerun button is a loan shark

Re-running failed jobs in GitHub Actions has existed since March 2022, and it is genuinely useful for infrastructure hiccups — a runner lost its network for thirty seconds, that sort of thing. The problem is what it does to your habits when the failure is a flaky test, not a flaky runner.

    gh run rerun 123456789 --failed

One command, green build, zero learning. I counted once: in a single week our team reran the suite eleven times and fixed zero tests. Each rerun cost ten to twenty minutes plus a context switch, and the flake was still there on Monday, collecting interest. Retries are a shock absorber, not a repair strategy. If a test needs a retry to pass, that retry-and-pass is itself flake data — record it, do not celebrate it.

My rule now: a single rerun to confirm suspicion is fine. The second rerun for the same test identity without a ticket is a process failure, not bad luck. Cap automatic retries at one or two — Maven Surefire even has the knob (`rerunFailingTestsCount`), and I leave it at one precisely so the suite stays honest — and feed every retry-and-pass back into your flake tracking.

### Quarantine is a hospital, not a hospice

GitLab's quarantine process, which has been public in their handbook for years, got the framing right: quarantine temporarily removes a flaky test from the blocking path while keeping it running, visible, owned, and on a deadline. The test still executes on every run. Its failure just stops blocking everybody else's merge.

Here is the version that finally worked for us, kept deliberately small:

- **Same code, different result = flaky by definition.** A test that passes and fails on the same commit SHA is flaky. That is the only detection rule you need to start — JUnit XML already gives you per-test results, so compare across runs before you argue about it.
- **Quarantine means non-blocking, never deleted.** We keep a `test-quarantine.yml` file in the repo listing the test, the owner, the ticket, and an expiry date. Changing it goes through normal code review, so quarantine is a visible risk acceptance, not a silent skip.
- **Quarantined tests still run in a shadow lane.** Ours run on every main build plus the nightly full suite, non-blocking. If you stop running them you lose the evidence you need to bring them back — and you will never notice when the underlying code drifts further.
- **Every entry has an owner, a ticket, and an expiry.** Two weeks, then it either rejoins the blocking suite with evidence or gets formally retired with written reasoning. No expiry, no quarantine. That single rule is what keeps the list from becoming a junk drawer.

The failure mode to avoid is the `@Disabled` annotation that lives forever. That is not quarantine, that is deletion with extra steps and false comfort. Six months later you have forty "temporarily" skipped tests and a coverage report that lies to you.

### Retry budgets that actually work

Once the policy exists, the mechanics get simple. Our pipeline has two stages: the blocking suite (stable tests — a failure here stops the merge, full stop) and the quarantine lane (known-flaky tests — they run, results are recorded, failures annotate the PR but do not block it). The deploy job only looks at the blocking suite.

A few details that mattered more than I expected:

- **Do not quarantine on a single PR failure.** Require history — say, a 20% flake rate over the last thirty main runs with at least a handful of data points. A deterministic failure that is new in your PR is a blocker, not a flake, even if you really want it to be a flake.
- **Quarantine the test, not the whole job.** The one time we quarantined an entire integration-test job, we hid a real regression inside it for a week. Split at the test level so the blast radius stays small.
- **Track the count as a metric.** Ten quarantined tests added yesterday is less worrying than two tests sitting untouched for ninety days. Age is the better signal. We review the list in the same weekly slot as incidents until it shrinks.
- **Re-promotion needs evidence, not optimism.** Our bar: twenty consecutive clean shadow runs plus a root-cause note in the ticket, then a removal PR. "It passed once on my laptop" does not count. After rejoining, we watch it for a week and re-quarantine immediately on relapse, no shame in it.

Most of our flakes were the usual suspects: a fixed `Thread.sleep` waiting for an async settlement callback, two tests writing to the same account row under parallelism, and one test hitting a real sandbox endpoint over the network. Boring causes, boring fixes — per-test transactional fixtures, unique test data per test, mock the external call. The quarantine list did not fix any of that. It just bought us the quiet to fix it without blocking the team every day.

### Owning the suite again

Three weeks later the list went from nine entries to two, and something subtle changed: people started reading failures again. When the build went red, it meant something, because the known noise was labelled and fenced off instead of smeared across every PR. The two remaining entries each had a name, a ticket, and a date — which meant they were work, not weather.

If I had to pick one habit that rebuilt trust in our pipeline, it would be this: never let a flaky test fail anonymously twice. The first failure gets investigated, the repeat gets quarantined with an owner and an expiry, and the rerun button goes back to being what it was meant for — the occasional infrastructure sneeze, not daily medication. A green build nobody believes is worse than no build at all. Say no to the noise on purpose, and the signal comes back.
