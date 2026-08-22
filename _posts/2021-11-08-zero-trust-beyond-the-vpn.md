---
layout: post
title:  "Zero Trust - The VPN Is Not the Castle Wall"
date:   2021-11-08 18:42:00
comments: True
categories: [Software, Security]
excerpt_separator: "<!--more-->"
---

Security still shows up in architecture reviews as a network diagram with a thick line around "us" and a thinner one around "them." VPN on, you are inside. Inside, the payment API will talk to the reporting API because they share a subnet and a long-lived service account someone checked in years ago.

That model has been fraying for a while — cloud, contractors, WFH laptops that never see the office again — and the slide decks all say "zero trust" now. I have been trying to translate the buzzword into things a .NET team can actually change without waiting for a five-year platform programme.

<!--more-->

### What zero trust is not

It is not a product SKU. It is not "turn off the VPN and hope." And it is not only an identity-team problem for human logins.

NIST's [SP 800-207](https://csrc.nist.gov/publications/detail/sp/800-207/final) (2020) is still the cleanest definition I keep pointing people at: no implicit trust from network location or asset ownership; authenticate and authorize before a session to a resource; assume the network is hostile, including the one you think you own. Google's public write-ups on [BeyondCorp](https://cloud.google.com/beyondcorp) (users/devices) and [BeyondProd](https://cloud.google.com/blog/products/identity-security/applying-zero-trust-to-user-access-and-production-services) (workloads talking to workloads) make the same split in plainer English.

If your mental model stops at "MFA on the VPN portal," you have done half of BeyondCorp and none of BeyondProd. The breach path I keep seeing in postmortems is lateral movement *after* something already had a foothold inside the perimeter.

I think of the old castle wall like a nightclub rope. Once you are past the bouncer, every room assumes you belong. Zero trust is more like a hotel key card: every door checks again, and the card only opens the floors you paid for.

### Three places trust still hides in our apps

When I walk an ASP.NET Core service map with this lens, the same three smells show up:

- **Shared network = shared trust.** Service A can hit Service B's cluster IP because NetworkPolicy is "allow all in namespace" and nobody rotated the internal basic-auth header since 2019.
- **Bearer tokens treated like room keys for life.** A JWT stolen from a browser log or a debug proxy works from any IP until expiry. We already know bearer is cash — anyone holding it spends it — but internal APIs often skip even short TTLs and audience checks.
- **"Internal" endpoints with no caller identity.** Health-style admin routes, debug dumps, or "temporary" sync APIs bound on all interfaces because "it is only on the cluster network."

None of that needs a new vendor. It needs fewer assumptions in the code and the mesh.

### What we are actually changing this quarter

We are not boiling the ocean. Three concrete moves, in order of pain vs payoff for our stack.

**1. Kill ambient trust between services.**

Where we already run a service mesh, turn on mutual TLS and stop treating plaintext inside the cluster as fine. Istio has been doing [auto mTLS](https://istio.io/latest/docs/tasks/security/authentication/authn-policy/) for a while: sidecars upgrade pod-to-pod traffic when both sides are in the mesh, and you graduate from `PERMISSIVE` (accept plaintext *and* mTLS) to `STRICT` when the stragglers are gone. A mesh-wide or namespace `PeerAuthentication` is the policy lever; the app code should not be inventing its own TLS handshake for every pair of services.

If you do not have a mesh yet, start smaller: TLS on the east-west path you control, and **require a client certificate or a short-lived workload identity** on the two or three APIs that can move money or PII. ASP.NET Core has had [certificate authentication](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/certauth) for years — `AddCertificate`, validate chain and subject, map to a claims principal. It is fiddly with load balancers that terminate TLS (you must forward the client cert and the proto correctly), but for true service-to-service it is honest proof of *who* is calling, not just *that* someone has a network path.

**2. Make every request answer "who" and "why this."**

For user-facing APIs we already validate JWTs. For service calls we still have too many static keys in Kubernetes secrets. The direction of travel: workload identity or short-lived tokens with a tight **audience** claim, checked on every request — not a subnet ACL and a prayer. Least privilege means the reporting job's credential cannot call the payout endpoint even if DNS resolves.

I keep a one-line rule on the wiki: *if we cannot name the caller in the logs, we cannot revoke the caller after an incident.*

**3. Continuous, not one-time, access decisions.**

Device posture and session risk for humans is mostly an IdP / conditional-access problem. On the app side, the equivalent is: do not mint a god-token at login and bless every microservice for eight hours. Prefer step-up for sensitive actions, short access-token lifetimes, and refresh that can be killed server-side. Pair that with the correlation IDs and structured logs we already invested in so "who called what" is reconstructable when something looks wrong.

### Mistakes I want us to avoid

- **Big-bang "zero trust project."** You will drown in diagrams. Pick one trust boundary (one namespace, one payment path) and make lateral movement harder there.
- **mTLS theatre.** Encrypting garbage between two over-privileged identities is still garbage. Identity and authorization policies matter as much as the TLS tickbox.
- **Excluding break-glass.** Emergency access will exist. Document it, monitor it, time-box it. Pretending it does not exist is how permanent backdoors get named "temporary."
- **Forgetting developers.** If local and CI cannot authenticate the same way prod does (even with test CAs and mocked identity), people will carve permanent holes "just for debugging."

### My take

Zero trust is not a replacement for patching, supply-chain hygiene, or not checking secrets into Git. Those still matter — maybe more, because you are admitting the perimeter will fail.

What it *does* change is the default question in design review. Not "is this inside the VPC?" but "what proves this caller is allowed to do *this* action *right now*, and how do we turn that off?"

If you only do one thing this month: pick your highest-value internal API, require an explicit caller identity (mesh mTLS identity, client cert, or short-lived token with audience), deny the rest, and watch the logs for a week. The first unexpected caller is usually more educational than the strategy deck.

The castle wall made us lazy. Key cards on every door are annoying until the day someone is already inside — and you still get to lock the vault.
