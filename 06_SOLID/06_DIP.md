# Dependency Inversion Principle

High-level business logic should depend on abstractions rather than concrete infrastructure details. This isolates the core domain from volatile external dependencies like databases, third-party APIs, and message brokers.

## Application to SALESTORM

### Payment Flow
```text
PaymentService (High-level policy)
    |
    v
PaymentProvider (Abstraction)
    |
    +---- MockPaymentProvider (Low-level detail)
    +---- ExternalPaymentProvider (Low-level detail)
```

### Inventory Flow
```text
ReservationService (High-level policy)
    |
    v
InventoryRepository (Abstraction)
    |
    v
PostgreSQL implementation (Low-level detail)
```

### Order Flow
```text
OrderService (High-level policy)
    |
    v
OrderRepository (Abstraction)
```

## Why Dependency Inversion Matters

- **Unit testing:** We can run the 14 automated test scenarios without requiring a live Stripe endpoint or Kafka cluster by injecting mock implementations of these abstractions.
- **Easier repository replacement:** Data access logic is hidden behind the abstraction.
- **Easier mocking:** High-level services are tested in complete isolation.
- **Reduced coupling:** Infrastructure changes (e.g., updating a database driver) do not require changes to the business rules.
- **Clear business/infrastructure boundary:** The domain model remains pure.

## Architectural Note on Databases

While DIP allows us to abstract the database, **PostgreSQL remains the transactional source of truth** for core inventory and order data, as defined by the architecture. This principle does not imply that Redis replaces the transactional database; rather, the abstraction simply allows the application to remain agnostic to the exact SQL queries executed by the PostgreSQL repository implementation. Redis is strictly utilized as supporting infrastructure (admission control, deduplication fast-paths) and not the authoritative inventory store.
