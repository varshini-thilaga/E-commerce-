# Reliability and Failure Handling

This document defines how SALESTORM handles distributed failures predictably without cascading system collapse.

## Core Reliability Mechanisms

1. **Timeouts:** Every outbound network call (e.g., to the Payment Provider) has a strict, bounded timeout.
2. **Retries:** Transient failures (e.g., connection reset) are retried.
3. **Exponential backoff:** Retries are spaced out exponentially (e.g., 1s, 2s, 4s) to avoid overwhelming the dependency.
4. **Jitter:** Randomness is added to backoff times to prevent "thundering herd" retry storms.
5. **Maximum retry count:** Retries are strictly bounded (e.g., max 3 attempts).
6. **Circuit breaker:** Stops outbound calls entirely if a dependency is consistently failing, preventing thread exhaustion.
7. **Graceful degradation:** If a non-critical component fails, the system continues operating with reduced functionality.
8. **Fail-fast behavior:** Permanent failures (e.g., 400 Bad Request) are immediately failed without retrying.
9. **Backpressure:** Queues push back on producers when they are full, triggering load shedding upstream.
10. **Compensation:** Reverting previous business operations (e.g., releasing a reservation if payment fails).
11. **Reconciliation:** Background jobs that resolve ambiguous states (e.g., unknown payment outcomes).
12. **Health checks:** Probes to dynamically route traffic away from dead nodes.
13. **Recovery workers:** Background processes that replay stuck events.

## Important Rule: Do Not Retry Everything
We explicitly differentiate between transient failures and permanent failures. We **do not create retry storms**. Transient failures use bounded exponential backoff and jitter. Permanent failures fail fast. A Circuit Breaker does **not** replace timeouts or retries; it acts as a higher-level structural protection when retries are consistently failing.

## Failure-Handling Matrix

| Failure | Immediate Response | Recovery |
| --- | --- | --- |
| **Payment timeout** | Bounded retry / unknown outcome handling | Reconciliation |
| **Payment failure** | Release reservation | Compensation |
| **Order Service down** | Persist durable event | Replay/recovery |
| **Database failure** | Fail safely | Database recovery |
| **Event consumer failure** | Retry | DLQ (Dead Letter Queue) |
| **Traffic overload** | Admission rejection (HTTP 429) | Continue accepting when capacity returns |
