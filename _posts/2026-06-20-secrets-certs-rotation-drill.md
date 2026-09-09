---
layout: post
title:  "Secrets and Certs - Rotation Is a Drill, Not a Ticket"
date:   2026-06-20 10:30:00 +0530
comments: True
categories: [Software, Security]
excerpt_separator: "<!--more-->"
---

The staging environment died on a Saturday morning, and for twenty minutes everyone was sure it was a bad deploy. It was not a bad deploy. Friday evening someone had pushed a perfectly good build, and Saturday morning every call to one Spring Boot service started failing TLS handshakes. The certificate on the ingress had expired at midnight, quietly, while we were all asleep.

I have seen the long-lived database password version of the same movie too. A credential that has not changed in two years, pasted into three config files and one wiki page, and rotating it feels like defusing something because nobody knows what will break. Both outages taught me the same lesson: rotation is not a ticket you handle when it fires. It is a drill you practice until it is boring.

<!--more-->

### The Saturday the cert died

Our staging cluster back then ran cert-manager, so in theory this could not happen. In practice, theory had a gap. The Certificate resource for that ingress pointed at an issuer that someone had renamed during a cleanup, and the renewal failures had been sitting in `kubectl describe certificate` for weeks where nobody looked. cert-manager retries with backoff and re-issues when the expiry approaches or when the SANs or issuer drift, but it cannot fix a reference that no longer resolves. The cert sat there, valid until it was not.

The fix took ten minutes once we saw it. The embarrassment lasted longer, because the monitoring gap was entirely ours. We had alerts for pod restarts and error rates, and nothing that watched `status.notAfter` on our certificates. A one-line custom column would have told us a month earlier:

```
kubectl get certificate -o custom-columns=NAME:metadata.name,EXPIRES:status.notAfter
```

That command is now in our runbook, and the expiry date is on a dashboard next to the error budget. Certificates tell you exactly when they will betray you. It is rude not to listen.

### Secrets should have a short life

Database passwords are the other half of the same habit. For years my default was a long-lived password stored in Vault's KV engine, synced into the cluster, mounted as an environment variable, and then left alone because touching it felt risky. The longer it lives, the scarier rotation gets, which means it lives even longer. That loop is the vulnerability.

What finally broke the loop for me was switching the services I own to short-lived dynamic credentials from Vault's database secrets engine. Instead of one password that lives for years, the app gets a lease that lives for hours and the Vault agent handles renewal. If a credential leaks, its blast radius has an expiry date. If rotation breaks, I find out on a Tuesday afternoon, not during an incident, because it happens all the time.

On the Kubernetes side I sync through the External Secrets Operator. One `ExternalSecret` per service, with a boring `refreshInterval` like an hour and the default periodic policy, so the target Secret is re-read and updated continuously:

```
spec:
  refreshInterval: 1h
  refreshPolicy: Periodic
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: payments-db-creds
```

A few things I learned the hard way:

- Keep one source of truth. Vault holds the credential; the cluster only holds a copy. Revoke or rotate in Vault and let the operator propagate it, instead of editing Secrets by hand in five namespaces.
- Know what consumes the Secret. An updated Secret does not restart your pods by itself. We roll the Deployment on rotation, through the operator's rotation support or a reloader, so the app actually picks up the new value.
- Practice the force-sync path before you need it. `kubectl annotate es <name> force-sync=$(date +%s) --overwrite` is the kind of command you want in muscle memory, not in a wiki page you read while paged.
- Give every secret an owner and an expiry story. If nobody can answer "what breaks if I rotate this right now," that is the finding, and it outranks the rotation itself.

In a bank none of this is optional decoration. Auditors love asking when a credential was last rotated and who can see it. "Automatically, every few hours, and here are the logs" ends that conversation fast.

### Certs renew themselves if you let them

cert-manager earns its keep the same way: by making renewal so routine that expiry stops being an event. The shape of it is simple. You declare a `Certificate` with a duration and a `renewBefore`, cert-manager talks to the issuer, and the signed cert plus private key land in a Secret your ingress or pod mounts:

- Defaults are saner than most people think: 90 days duration, renewal starting 30 days before expiry, or two-thirds through the lifetime if you leave `renewBefore` unset.
- It supports the issuers I actually meet: public ones like Let's Encrypt, private PKI, and Vault's PKI engine for internal services.
- When renewal fails it backs off and retries, and you can trigger one by hand with `cmctl renew` instead of waiting and hoping.

The operational part is the same as with secrets. Watch the expiry, alert on it with margin, and rehearse the failure. Our drill now is deliberately unglamorous: pick a staging cert with a short `renewBefore`, break the issuer reference on purpose, watch the alert fire, fix it, watch the `CertificateRequest` go through. Twenty minutes, once a quarter, and the Saturday-morning version never happens again.

### Make rotation a drill

Here is the checklist that stuck on our team wall, after the staging postmortem:

- Every secret has a TTL story: dynamic lease, auto-rotated value, or a calendar date with an owner. "Forever" is not a story.
- Every certificate has an expiry alert with at least two weeks of margin, routed to the team channel, not to one person's inbox.
- Rotation is rehearsed quarterly: rotate staging database credentials end to end, force-renew one cert, confirm the pods actually picked both up.
- The runbook fits on one page: where the source of truth lives, how to force a sync, how to roll the workload, who approves in production.

None of this is clever. That is the point. Secrets and certificates are the kind of infrastructure that fails exactly in proportion to how rarely you touch it. Touch it on purpose, on schedule, while everything is green, and it stops being scary.

My unpopular opinion, earned on that Saturday: if your rotation plan starts with "raise a ticket when it expires," you do not have a rotation plan. You have a future outage with a date already printed on it. Shorten the lifetimes, automate the renewal, and practice until the drill is the most boring meeting of the quarter.
