---
layout: post
title:  "Circuit Breakers - Stop Calling a Service That Is Already On Fire"
date:   2023-07-16 11:34:00
comments: True
categories: [Architecture]
excerpt_separator: "<!--more-->"
---

A few weeks into Spring Boot land and I already have a new favourite kind of production pain: one slow downstream dependency quietly taking the whole request path with it. Not a crash. Not a clean 500. Just threads waiting, timeouts stacking, and suddenly *your* service looks unhealthy because someone else's database is having a bad afternoon.

In the .NET world I reached for Polly when this showed up. In Java the answer that keeps coming up in code reviews is Resilience4j — the library that basically replaced Netflix Hystrix after Hystrix went into maintenance mode back in 2018. If you are wiring Spring Boot 3 services that talk to other services (and who isn't), this is the pattern I am finally treating as non-optional.

<!--more-->

### The fuse box, not the fire extinguisher

A circuit breaker is not magic retry fairy dust. Think of the electrical panel in your house. When something shorts, the breaker *opens* so the rest of the house does not burn. After a cool-down it lets a little current through (half-open). If things look fine, it closes again. If not, it trips open once more.

Resilience4j models that almost literally:

- **Closed** — calls flow; failures are counted in a sliding window
- **Open** — calls fail fast (or hit a fallback); the sick dependency is left alone
- **Half-open** — a limited number of probe calls decide whether to close or re-open

That fail-fast part is the whole point. A 50ms "nope, circuit open" is infinitely kinder to your thread pool than a 30-second socket hang that multiplies across every concurrent user.

### Why Hystrix stories still matter in 2023

Hystrix taught a generation of Java teams the vocabulary — bulkheads, fallbacks, dashboards. Then Netflix put it in maintenance mode, Spring Cloud stopped treating it as the happy path, and Resilience4j became the default recommendation. Lightweight, modular, Java 8+ functional style, and it has a dedicated `resilience4j-spring-boot3` module so you are not fighting Boot 3 / Jakarta packaging with an old Boot 2 starter.

I care less about the brand name and more about the contract: decorate the call, configure thresholds in config (not buried in tribal knowledge), expose state through Actuator/Micrometer so on-call can *see* the breaker trip instead of guessing from latency charts.

### What I actually put on a service method

The annotation path is what most Spring code I am reading uses. Rough shape:

```java
@CircuitBreaker(name = "paymentsClient", fallbackMethod = "paymentsFallback")
@Retry(name = "paymentsClient")
public PaymentStatus getStatus(String paymentId) {
    return paymentsClient.fetch(paymentId);
}

private PaymentStatus paymentsFallback(String paymentId, Throwable t) {
    log.warn("payments circuit/fallback for id={}", paymentId, t);
    return PaymentStatus.unknown(paymentId);
}
```

And the knobs live in `application.yml` where reviewers can argue about them:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentsClient:
        slidingWindowSize: 20
        minimumNumberOfCalls: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
  retry:
    instances:
      paymentsClient:
        maxAttempts: 3
        waitDuration: 200ms
        enableExponentialBackoff: true
```

Those numbers are starting points, not gospel. Copy-pasting someone else's `failureRateThreshold: 50` without knowing your traffic shape is how you either never open (too timid) or flap open on every blip (too twitchy).

### The mistakes I keep seeing (and making)

**Retry *outside* a breaker without thinking.** Retry storms are real. If fifty instances each retry a dying dependency three times, you just DDoS'd the patient. Prefer: timeout on the HTTP client, then breaker, then careful retry only on idempotent reads — or retry *inside* a budget the breaker already limits.

**Fallbacks that lie.** Returning an empty list because payments is down might be fine for a banner. It is not fine if the next step books a trade. Degraded responses need to be honest in the API contract and visible in metrics. Silent wrong answers are worse than loud failures.

**Breakers on the wrong boundary.** Protect *outbound* calls to things you do not control — HTTP clients, message brokers, flaky partner APIs. Wrapping your own pure in-memory logic in a circuit breaker is ceremony without benefit.

**No one watching state.** If Actuator health or Micrometer metrics for the breaker are not on a dashboard, you will learn the circuit opened from a customer, not from a graph. Boot + Micrometer makes this cheap; use it.

**Thread pool bulkheads "because Hystrix did".** Resilience4j defaults lean semaphore bulkheads; thread-pool isolation is opt-in and costs threads. Start simple. Add isolation when one dependency's latency profile is poisoning shared executors.

### How this maps from my Polly mental model

| Polly idea | Resilience4j cousin |
| --- | --- |
| `CircuitBreakerPolicy` | `CircuitBreaker` |
| `WaitAndRetry` | `Retry` |
| `Timeout` | `TimeLimiter` (+ client timeouts) |
| Bulkhead / isolation | `Bulkhead` |
| Fallback delegate | `fallbackMethod` / `Decorators` recover |
| Policy wrap order | Annotation order + decorator chain — be deliberate |

Same job, different dialect — which seems to be the theme of my year.

### What I am doing on the next service I touch

1. Inventory outbound calls. Anything over the network gets a name in config.
2. Set client timeouts first. A breaker without timeouts is a seatbelt with no brakes.
3. Add a circuit breaker with conservative windows; tune after a week of metrics.
4. Fallbacks only where product accepts degradation; otherwise fail fast and let the caller decide.
5. Alert on open state and on elevated fallback rates, not only on p99 latency.

I am not sprinkling `@CircuitBreaker` on every method like confetti. I am putting it on the two or three dependencies that have already burned us.

### My take

Distributed systems fail in boring ways: slow, partial, and at 2 a.m. Circuit breakers do not fix the dependency. They stop your service from volunteering as a sacrifice.

If you are on Spring Boot 3 in 2023 and still hoping retries and bigger thread pools will save you, try the fuse box instead. Resilience4j is not glamorous. It is the difference between one red dependency and a SEV that pages half the org because every pod was politely waiting forever. I would rather return a clear degraded response in milliseconds than be the architect explaining why we melted ourselves on someone else's outage.
