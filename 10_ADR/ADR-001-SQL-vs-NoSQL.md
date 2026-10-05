# ADR-001: SQL vs NoSQL for Core Transactions

## Status

Accepted

## Context

SALESTORM requires a highly reliable datastore to track inventory, reservations, orders, payments, and customers. During a flash sale, 10,000 requests will compete for the exact same 100 inventory units. We must select a primary database paradigm that strictly guarantees invariants (e.g., preventing negative inventory).

## Decision

Use PostgreSQL (SQL/Relational) as the transactional system of record for core relational business data.

## Alternatives Considered

**NoSQL-first architecture (e.g., MongoDB, Cassandra, DynamoDB).**
- *Pros:* Massive horizontal read/write scalability, flexible schemas, easy geographical distribution.
- *Cons:* Weaker ACID transaction support across multiple documents/tables, lack of strict schema enforcement, and eventual consistency (in some systems) makes strictly preventing a `0 - 1` inventory condition vastly more complicated at the application layer.

## Trade-offs

**What do we gain?**
- Strict ACID transactions.
- Referential integrity (Foreign Keys between Reservations, Payments, and Orders).
- The ability to use database-level constraints (e.g., `CHECK (available_quantity >= 0)`) to mathematically prevent overselling.
- Atomic conditional updates.

**What do we give up?**
- Boundless horizontal write scalability. PostgreSQL is traditionally scaled vertically for writes, meaning single-row contention becomes the primary bottleneck during a flash sale.

## Consequences

**Positive:**
- Complete confidence in inventory correctness and financial data consistency.

**Negative:**
- We must aggressively protect the database using upstream admission control and caching, as we cannot simply add more database write shards to handle the sudden burst of 10,000 requests hitting a single inventory row.

## Revisit Conditions

This decision should be reconsidered if the product catalog grows so massive that a single relational cluster can no longer contain it, requiring a transition to a distributed NewSQL system (e.g., CockroachDB) or a strictly sharded NoSQL approach. 

*Important: Redis may still be used alongside PostgreSQL for caching and fast-path admission control, but Redis is NOT the source of truth for inventory.*
