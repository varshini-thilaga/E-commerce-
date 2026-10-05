# ADR-002: Sync vs Async Communication

## Status

Accepted

## Context

SALESTORM must process user requests rapidly while interacting with multiple distinct domains (Inventory, Payment, Order). We must decide how these domains communicate during the critical path of a purchase.

## Decision

Use a hybrid communication model.

**Synchronous for critical request/response operations:**
- Product catalog reads where appropriate.
- Reservation requests (so the user knows immediately if they got the item).
- Checkout coordination.
- Payment initiation.
- Immediate API responses confirming the transaction intent.

**Asynchronous for eventual consistency and background work:**
- Order recovery and creation after payment success.
- Notifications (email/SMS).
- Durable event propagation.
- Retry/replay workflows.

## Alternatives Considered

1. **Fully Synchronous:**
   - *Pros:* Simple to reason about; the client waits until everything (including order creation and emails) is done.
   - *Cons:* High latency. If the Order Service is down or slow, the Payment and Reservation services crash or time out, violating our resilience goals and increasing tight coupling.

2. **Fully Asynchronous:**
   - *Pros:* Maximum resilience and throughput.
   - *Cons:* Highly complex user experience. If reservation is async, the user clicks "Buy" and must poll to find out if they actually secured the item, which is unacceptable for a high-tension flash sale.

## Trade-offs

**What do we gain?**
- We balance responsiveness (the user gets immediate confirmation of reservation and payment intent) with resilience (downstream failures do not break the upstream user experience).

**What do we give up?**
- Architectural complexity. We must maintain both REST APIs and message queues/event brokers, requiring correlation IDs and careful state tracking.

## Consequences

**Positive:**
- The system can survive an Order Service outage (demonstrated in the prototype via a 30-second outage recovery).

**Negative:**
- The frontend client must be designed to handle pending states (e.g., "Payment accepted, processing order") while waiting for asynchronous completion.

## Revisit Conditions

This decision should be reconsidered if business requirements change to demand immediate, guaranteed fulfillment confirmation before charging the user, which would force a shift back toward distributed synchronous transactions (e.g., Saga orchestration).
