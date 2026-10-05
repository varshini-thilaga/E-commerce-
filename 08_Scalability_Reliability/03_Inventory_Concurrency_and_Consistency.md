# Inventory Concurrency and Consistency

*This is the MOST IMPORTANT document defining how SALESTORM prevents overselling.*

## The Problem
During a flash sale, 10,000 concurrent requests compete for 100 available units. The system must guarantee the core invariant:
`available_quantity >= 0`

If multiple threads read the available quantity as `1` simultaneously, and all decrement it, the quantity drops below zero (overselling).

## The Reservation Transaction (Conceptual)
The consistency boundary is maintained at the database level.

```sql
BEGIN TRANSACTION;

-- Validate request & create reservation record
INSERT INTO reservations (id, user_id, product_id, status) VALUES (...);

-- Atomically verify and reduce quantity
UPDATE inventory
SET available_quantity = available_quantity - :quantity,
    reserved_quantity = reserved_quantity + :quantity,
    version = version + 1
WHERE inventory_id = :inventory_id
  AND available_quantity >= :quantity;

COMMIT;
```
*(Note: This is conceptual SQL. If the UPDATE affects 0 rows, the transaction is manually rolled back in code, rejecting the reservation due to insufficient stock).*

## Concurrency Approaches

### Approach 1: Pessimistic Locking
- **Mechanism:** `SELECT ... FOR UPDATE`. Locks the row when reading.
- **Correctness:** High.
- **Contention:** Very High. All 10,000 requests queue up sequentially waiting for the lock.
- **Throughput:** Low.
- **Failure behavior:** If a thread dies while holding the lock, the row remains blocked until timeout.

### Approach 2: Optimistic / Atomic Conditional Update (Selected)
- **Mechanism:** Using a `version` column (Optimistic) or a `WHERE available_quantity >= :quantity` clause (Atomic Conditional Update).
- **Correctness:** High. The database engine guarantees the atomic evaluation of the WHERE clause.
- **Contention:** Lower. Threads don't hold read locks.
- **Throughput:** High.
- **Failure behavior:** Fails fast. If the condition isn't met, it immediately returns 0 updated rows.

**Why we selected Approach 2:** It provides the strict ACID correctness required without the catastrophic lock contention and deadlocks associated with pessimistic locking during extreme flash sale bursts.

## Lifecycle Management
- **Reservation creation:** Atomically moves stock from `available` to `reserved`.
- **Reservation confirmation:** Upon successful payment, stock moves from `reserved` to `sold`.
- **Reservation expiry:** A background worker identifies unpaid reservations past their TTL.
- **Reservation release (Payment failure compensation):** If a payment fails or a reservation expires, a compensating transaction atomically moves stock from `reserved` back to `available`.

## Idempotency and Duplicates
If the same duplicate reservation request arrives twice, a unique constraint on the `(user_id, idempotency_key)` in the reservations table prevents the second request from initiating the inventory UPDATE, thereby preventing double-reserving.

## Important Architectural Rule
**Do not claim Redis alone can guarantee inventory correctness.**
While Redis (e.g., LUA scripts) can rapidly decrement a counter for fast-path admission, **PostgreSQL remains the transactional source of truth.** Redis is volatile and memory-bound; authoritative financial and inventory state must reside in the ACID-compliant relational database.
