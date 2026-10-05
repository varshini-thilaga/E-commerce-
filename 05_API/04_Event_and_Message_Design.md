# SALESTORM - Event and Message Design

This document details SALESTORM's asynchronous event model, crucial for achieving horizontal scalability and resilience against partial system outages.

## 1. Commands vs. Events

- **COMMAND:** An instruction asking a service to perform an action. It is imperative and can be rejected (e.g., due to validation or state).
- **EVENT:** A statement that something has already happened. It is a historical fact and cannot be altered or rejected.

---

## 2. Representative Commands
- `ReserveInventory`
- `ConfirmReservation`
- `ReleaseReservation`
- `ProcessPayment`
- `CreateOrder`
- `CreateShipment`
- `SendNotification`

## 3. Representative Events
- `InventoryReserved`
- `InventoryReservationConfirmed`
- `InventoryReservationReleased`
- `InventoryReservationExpired`
- `PaymentSucceeded`
- `PaymentFailed`
- `PaymentTimedOut`
- `OrderCreated`
- `OrderCreationRequested`
- `OrderCancelled`
- `ShipmentCreated`
- `NotificationRequested`

---

## 4. Event Envelope Structure

Events are wrapped in a standardized JSON envelope to facilitate uniform routing, tracing, and schema evolution.

```json
{
  "eventId": "evt-123",
  "eventType": "PaymentSucceeded",
  "eventVersion": 1,
  "occurredAt": "2026-01-01T00:00:00Z",
  "producer": "PaymentService",
  "correlationId": "corr-123",
  "aggregateId": "order-123",
  "payload": {
    "paymentId": "pay-123",
    "amount": 99.99
  }
}
```

### Envelope Fields:
- `eventId`: Globally unique identifier for the event instance.
- `eventType`: The business name of the event.
- `eventVersion`: Schema version for backward compatibility.
- `occurredAt`: Timestamp of creation.
- `producer`: The service that emitted the event.
- `correlationId`: Identifier linking all operations resulting from a single initial request.
- `aggregateId`: The primary domain entity ID (e.g., order ID) the event pertains to.
- `payload`: The event-specific data payload.

---

## 5. Event Documentation Example

### Event: `PaymentSucceeded`
- **Producer/Owner:** Payment Service
- **Consumers:** Order Service, Inventory Service, Notification Service
- **Purpose:** Indicates that funds have been successfully captured.
- **Required fields:** `paymentId`, `reservationId`, `amount`, `transactionReference`
- **Idempotency expectation:** Consumers MUST be idempotent.
- **Ordering considerations:** Should be processed before `OrderCreated`.
- **Retry behavior:** Exponential backoff.
- **Failure handling:** After max retries, route to Dead Letter Queue (DLQ).

---

## 6. Critical Recovery Scenario

A core resilience requirement of the flash sale is preventing order loss if downstream systems crash during peak load.

```text
Payment succeeds
        |
        v
Order Service unavailable
        |
        v
Durable event retained (e.g., in Message Broker or Outbox)
        |
        v
Order Service recovers
        |
        v
Event replay/reprocessing
        |
        v
Order created
```

---

## 7. Delivery Semantics & Idempotency

The messaging infrastructure guarantees **at-least-once delivery with idempotent consumers**. 

We do not claim exactly-once delivery, as true exactly-once semantics are prohibitively expensive and complex in distributed systems. Because events may be retried or redelivered by the message broker during network partitions or timeouts, **all event consumers must tolerate duplicate delivery.**

### Dead Letter Queue (DLQ) Behavior
If an event consumer repeatedly fails to process an event (e.g., due to a malformed payload or a hard database constraint violation), the event is moved to a Dead Letter Queue after a configured number of retries. 
This prevents "poison pill" messages from blocking the queue. DLQ messages trigger alerts for manual operator intervention and analysis.
