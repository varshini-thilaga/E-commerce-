# ADR-005: Inventory Consistency

## Status

Accepted

## Context

During a flash sale, 10,000 concurrent purchase attempts for 100 available units must never produce more than 100 successful reservations/sales. This is the most critical invariant of the SALESTORM project. Admission control sheds excess load, but it does not guarantee logical correctness. Redis can provide fast approximations, but is volatile.

## Decision

Protect inventory using transactional atomic conditional updates plus reservation lifecycle management and idempotency.

**The invariant:** `available_quantity >= 0`

## Alternatives Considered

1. **Admission Control Only:**
   - *Cons:* If we allow 100 requests through the gate, and 2 of them are duplicates from the same user, we might only sell 99 items, or race conditions might still oversell if the gate isn't perfectly synchronized. Admission control is a shield, not a correctness guarantee.
2. **Redis-Only Inventory:**
   - *Cons:* If Redis goes down or evicts keys, the source of truth is lost.

## Trade-offs

**What do we gain?**
- Absolute mathematical certainty. By coupling an atomic conditional update (`WHERE available >= quantity`) with a unique `Idempotency-Key` constraint in the database, we guarantee that the transaction remains the correctness boundary.

**What do we give up?**
- Database write capacity becomes the absolute limit of the system's throughput.

## Consequences

**Positive:**
- Zero overselling under any tested load.
- Safe lifecycle management: Reservation creation (atomic decrement), Reservation confirmation (payment success), Reservation expiry (TTL timeout), and Reservation release (payment failure compensation) are all handled predictably in the relational domain.

**Negative:**
- We must rely on background workers to clean up expired reservations, ensuring stock is eventually released back to the pool if users abandon their carts.

## Revisit Conditions

Revisit if the system architecture changes to an Event Sourcing paradigm where inventory is calculated as a projection of a stream of events rather than a mutable database row.
