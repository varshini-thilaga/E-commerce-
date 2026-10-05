# Circuit Breaker Pattern - Dependency Protection

## Problem
When a dependent external service (such as an external payment provider) becomes slow or unresponsive, attempting to call it continuously will cause threads, connections, and memory to pile up. This leads to resource exhaustion and cascading failure across the entire SALESTORM platform.

## Application
The Circuit Breaker pattern is applied primarily to external/dependent network calls to protect the system.

## Conceptual States

```text
CLOSED (Normal operation)
   |
   | repeated failures exceed threshold
   v
OPEN (Failing fast)
   |
   | recovery timeout elapsed
   v
HALF_OPEN (Testing recovery)
   |
   +---- success ---> CLOSED
   |
   +---- failure ---> OPEN
```

## Explanation

- **CLOSED:** Requests flow normally to the external provider.
- **OPEN:** Calls are rejected/failed fast immediately without attempting the network call, protecting the system from hanging threads.
- **HALF_OPEN:** After a timeout, limited test requests are permitted through to determine whether the dependency has recovered.

## Mechanisms
- **Timeout:** Maximum time allowed for a single call.
- **Retry & Backoff:** Handling transient network blips safely.
- **Circuit breaking:** Tripping the breaker after $N$ failures.
- **Metrics:** Continuous monitoring of the failure rate to determine state.

## Circuit Breaker vs. Idempotency

It is crucial to understand that a Circuit Breaker **does not replace idempotency**. 
They solve entirely different problems for payment operations:
- **Idempotency** protects against duplicate business operations (preventing double charging when retrying).
- **Circuit Breaker** protects the system resources from repeatedly calling an unhealthy dependency.
