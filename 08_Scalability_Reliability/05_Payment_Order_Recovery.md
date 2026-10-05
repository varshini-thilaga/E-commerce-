# Payment & Order Recovery

This document details the recovery workflow for a critical flash-sale failure scenario: **The payment succeeds, but the downstream Order Service is unavailable for 30 seconds.**

## The Scenario Flow

```text
Customer
   |
Checkout (Facade)
   |
Payment Service
   |
Payment Success (External Provider confirms charge)
   |
   v
Durable Event (PaymentSucceeded) published
   |
   X [Order Service unavailable - Connection Refused]
   |
   v
Event retained durably (Message Broker / Outbox)
   |
   v
Order Service recovers (30 seconds later)
   |
   v
Recovery Worker / Event Consumer pulls event
   |
   v
Order Created
```

## Explanation of Mechanisms

1. **Payment Idempotency:** Ensures that if the client retries the checkout, they are not charged a second time.
2. **Durable Payment Outcome:** The successful payment state is committed to the local database before downstream calls are attempted.
3. **Durable Event:** The system emits a `PaymentSucceeded` event to a durable message broker (like Kafka) or a transactional outbox table.
4. **Retry/Replay:** When the Order Service is down, the message broker retains the event and retries delivery based on a backoff policy.
5. **Idempotent Order Creation:** When the Order Service recovers and processes the event, it must be idempotent, guaranteeing it doesn't create two orders if the event is delivered twice.
6. **Recovery State:** The customer's reservation remains in a `CONFIRMED` state (not expired) while the order creation is pending.
7. **Reconciliation:** A background process can verify that all confirmed reservations eventually resulted in orders.
8. **Customer-visible status:** The UI informs the customer "Payment successful, processing order..." rather than showing a generic error.

## Why Not Synchronous Retries?

The system **must not** simply retry synchronously for 30 seconds while holding the original HTTP request open. Doing so would exhaust connection pools, tie up API Gateway threads, and cause the client browser to time out anyway. 

Instead, the synchronous request succeeds fast (returning a 202 Accepted or similar), and the asynchronous recovery mechanism (event replay) takes over the eventual consistency of the order state.

## Delivery Semantics
We do not claim exactly-once event delivery. The architecture guarantees **at-least-once delivery with idempotent consumers**.
