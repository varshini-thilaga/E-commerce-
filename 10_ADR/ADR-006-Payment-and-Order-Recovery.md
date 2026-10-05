# ADR-006: Durable Payment and Order Recovery

## Status

Accepted

## Context

If a customer's payment succeeds, but the downstream Order Service is unavailable (e.g., for 30 seconds), the system must not lose the order, nor should it force the user to wait indefinitely or retry (risking a double charge).

## Decision

Use idempotent payment processing plus durable events and asynchronous order recovery.

**Expected Design Flow:**
```text
Payment Success
    ↓
Durable Event (PaymentSucceeded)
    ↓
[Order Service unavailable]
    ↓
Event retained in Message Broker / Outbox
    ↓
[Order Service recovers]
    ↓
Replay / Recovery Worker
    ↓
Idempotent Order Creation
```

## Alternatives Considered

**Synchronous Retries:**
- *Pros:* Simple.
- *Cons:* Holding an HTTP request open for 30 seconds will cause the API Gateway and client browser to time out. The user might refresh the page and submit a duplicate payment request.

## Trade-offs

**What do we gain?**
- We protect the user experience and ensure financial integrity. The system acts with "at-least-once delivery with idempotent processing."

**What do we give up?**
- We give up strict immediate consistency. The customer's order might not appear in their "Order History" for a few seconds/minutes while recovery takes place.
- *Important:* We do NOT claim exactly-once delivery, as it is practically impossible in distributed messaging without massive overhead. We rely on idempotent consumers instead.

## Consequences

**Positive:**
- High resilience. Validated in prototype: 94 orders successfully recovered after a 30-second Order Service outage.

**Negative:**
- Requires complex reconciliation processes, Dead Letter Queues (DLQ), and robust idempotency keys.

## Revisit Conditions

Revisit if the payment provider introduces a strict requirement that fulfillment must be guaranteed *before* the capture of funds is finalized (requiring a two-phase commit or saga pattern).
