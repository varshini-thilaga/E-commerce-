# Message Processing and DLQ

This document defines the asynchronous processing standards for SALESTORM.

## The Event Flow

```text
Producer (e.g., Payment Service)
   |
   v
Durable Event Log / Queue (e.g., Kafka / RabbitMQ)
   |
   v
Consumer (e.g., Order Service)
   |
   +---- Success ----> Acknowledge (Message removed from queue)
   |
   +---- Temporary Failure ----> Retry (Message stays on queue, retried later)
   |
   +---- Repeated Failure ----> Dead Letter Queue (DLQ)
                              |
                              v
                         Investigation / Replay
```

## Core Principles

1. **Durable events:** Events are persisted to disk by the broker before they are acknowledged to the producer.
2. **Event producers:** Services that own a state change (e.g., Payment) emit the event.
3. **Event consumers:** Services that react to state changes (e.g., Order) consume the event.
4. **At-least-once delivery:** The broker guarantees delivery, but network partitions may cause an event to be delivered more than once.
5. **Idempotent consumers:** Because of #4, consumers MUST be idempotent. Do not claim exactly-once processing.
6. **Retry policy & Backoff:** Temporary failures (database lock timeouts, downstream 503s) trigger retries with exponential backoff.
7. **Dead Letter Queue (DLQ):** Messages that fail processing repeatedly (e.g., max 5 attempts) are routed to a DLQ.
8. **Poison messages:** Malformed payloads or unresolvable business errors (e.g., missing foreign keys) that will *never* succeed are "poison messages." The DLQ prevents these from blocking the main queue.
9. **Monitoring backlog:** High queue depth or DLQ growth triggers operator alerts.
10. **Recovery & Replay:** Operators can investigate DLQ messages, fix the underlying issue (or code bug), and manually replay them.

## Event Ownership Examples
- `PaymentSucceeded` (Owned by Payment)
- `OrderCreated` (Owned by Order)
- `InventoryReservationReleased` (Owned by Inventory)
- `InventoryReservationExpired` (Owned by Inventory)
- `NotificationRequested` (Owned by Order/System)
