# Observer Pattern - Events and Notifications

## Problem
In a decoupled microservice or modular architecture, the Payment Service needs to inform the Order Service when a payment succeeds. If the Payment Service calls the Order Service directly via a synchronous HTTP request, it creates tight coupling. If the Order Service is down, the Payment Service fails, risking a lost transaction. 

## Structure (Conceptual)

```text
EventPublisher
      |
      +---- OrderConsumer
      +---- NotificationConsumer
      +---- FulfilmentConsumer
      +---- RecoveryConsumer
```

## Examples of Published Events
- `PaymentSucceeded`
- `OrderCreated`
- `InventoryReservationReleased`
- `InventoryReservationExpired`

## Explanation

The Observer pattern flips the dependency. 
- The **Producer (Subject)** (e.g., Payment Service) emits an event but has no direct knowledge of who is listening.
- The **Consumers (Observers)** (e.g., Order Service, Notification Service) subscribe to events they care about and react accordingly.

This massively reduces coupling. It allows the future addition of new observers (like an Analytics service) without modifying the producer's code.

## Critical Production Considerations

**Important:** A classic, in-memory, synchronous implementation of the Observer pattern (like Java's `java.util.Observer`) DOES NOT provide durable messaging. If the application crashes while an observer is processing, the event is lost.

In SALESTORM, true resilience requires applying this pattern via a **durable event/message mechanism** (e.g., a Message Broker like Kafka or RabbitMQ, or a transactional Outbox pattern). This ensures that events survive crashes. Consequently, because network events can be retried, **all consumers must be strictly idempotent.**
