# Repository Pattern - Persistence Abstraction

## Problem
Business services need to persist data, but directly embedding SQL queries or ORM commands (like `session.commit()`) into business logic makes the code difficult to unit test and tightly couples the domain to a specific database technology.

## Structure (Conceptual Examples)

- `InventoryRepository`
- `ReservationRepository`
- `PaymentRepository`
- `OrderRepository`

```text
ReservationService
      |
      v
InventoryRepository (Abstraction)
      |
      v
PostgreSQL
```

## Benefits

- **Testability:** Repositories can be easily mocked to test complex business logic without a live database.
- **Separation of concerns:** Business rules are entirely separate from data access logic.
- **Database implementation isolation:** The domain remains agnostic to the specific SQL dialect or ORM in use.
- **Clear persistence boundary:** Centralizes all queries, making optimization and indexing easier to manage.

## Important SALESTORM Architectural Rules

In SALESTORM, **PostgreSQL remains the transactional source of truth** for core inventory and order data, as defined by the architecture. 

**Redis must not silently become the authoritative inventory database.** While Redis may be used for admission control or deduplication fast-paths, the Repository implementation for inventory relies on the relational database for ACID guarantees.

Furthermore, utilizing a repository abstraction **does not eliminate the need for correct transaction boundaries and concurrency control.** The repository must still correctly utilize database locks, atomic conditional updates (e.g., `UPDATE ... WHERE available >= quantity`), and isolation levels to prevent overselling.
