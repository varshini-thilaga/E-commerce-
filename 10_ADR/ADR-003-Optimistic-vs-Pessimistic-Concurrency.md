# ADR-003: Optimistic vs Pessimistic Concurrency

## Status

Accepted

## Context

During the flash sale, 10,000 concurrent requests target the same inventory record (100 units available). We must select a concurrency control strategy at the database layer to manage this extreme contention without deadlocking, crashing, or overselling.

## Decision

Use an atomic conditional database update as the primary inventory reservation mechanism.

Conceptually:
```sql
UPDATE inventory
SET available_quantity = available_quantity - :quantity,
    reserved_quantity = reserved_quantity + :quantity,
    version = version + 1
WHERE inventory_id = :inventory_id
  AND available_quantity >= :quantity;
```

## Alternatives Considered

1. **Pessimistic Locking (`SELECT FOR UPDATE`):**
   - *Pros:* Strictly serializes access; high correctness.
   - *Cons:* Disastrous under extreme contention. 10,000 requests queueing for a single lock will exhaust database connection pools, causing cascading timeouts and system failure.

2. **Strict Optimistic Concurrency (Version checking):**
   - *Pros:* No locks held during reads.
   - *Cons:* Under 10,000 concurrent requests, 9,999 transactions will read version 1, attempt to write version 2, and fail with a `StaleObjectStateException`. This requires the application layer to constantly retry, burning CPU and network bandwidth.

## Trade-offs

**What do we gain?**
- **Atomic Conditional Updates** shift the validation (`available_quantity >= quantity`) directly to the database engine at the exact moment of the write. The database natively handles the row lock for the microsecond of the `UPDATE`, without holding a long-running pessimistic lock across multiple application-level queries.
- We achieve high throughput and strict protection of the `available_quantity >= 0` invariant.

**What do we give up?**
- We lose the ability to read the exact state of the row *before* making complex application-level decisions, forcing the business logic to rely on the success/failure of the `UPDATE` statement (rows affected = 1 vs 0).

## Consequences

**Positive:**
- Survives 10k bursts without deadlocking. Fails fast when inventory is depleted.

**Negative:**
- The ORM/application logic must be explicitly designed to handle conditional update failures as standard business outcomes (e.g., "Sold Out") rather than generic exceptions.

## Revisit Conditions

Revisit if the business logic for allocating inventory becomes so complex (e.g., checking multiple different tables, user tiers, and geographic restrictions simultaneously) that a single atomic `UPDATE` statement can no longer express the logic, necessitating a shift to pessimistic locking or serialized queue processing.
