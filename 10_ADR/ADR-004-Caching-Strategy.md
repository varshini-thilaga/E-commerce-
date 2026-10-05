# ADR-004: Caching Strategy

## Status

Accepted

## Context

To handle intense traffic, we must shield the primary transactional database (PostgreSQL) from heavy read volumes and repetitive fast-path checks. However, relying too heavily on cache for transactional data risks corrupting the business state if the cache and database diverge.

## Decision

Use Redis primarily for non-authoritative fast-path data, employing a Cache-Aside strategy.

- **Redis is used for:** Product/catalog caching, hot sale metadata (start/end times), admission-control support (token buckets), and short-lived/idempotency-related data where appropriate.
- **PostgreSQL remains authoritative for:** Inventory, Reservations, Orders, and Payments.

## Alternatives Considered

**Write-Through / Cache-as-System-of-Record:**
- *Pros:* Extreme read/write throughput.
- *Cons:* Extremely dangerous for financial/inventory data. If Redis crashes before syncing to disk/PostgreSQL, data is lost, resulting in oversold inventory or missing orders.

## Trade-offs

**What do we gain?**
- Safely reduces read pressure on PostgreSQL by serving 99% of product catalog views from memory.
- Protects the transactional core.

**What do we give up?**
- Adds infrastructure complexity.
- Introduces cache invalidation challenges.

## Consequences

**Positive:**
- Cache failure (e.g., Redis restarting) must not and will not corrupt transactional inventory state. The system simply degrades gracefully, reading from the database directly (though likely triggering admission control if the database becomes overloaded).

**Negative:**
- We must handle TTL (Time-To-Live), stale data, and "cache stampedes" (when a hot key expires and thousands of requests simultaneously hit the database).

## Revisit Conditions

Revisit if the read load on PostgreSQL remains unsustainably high despite the cache-aside strategy, requiring more aggressive caching mechanisms like materialized views or write-through caches for non-critical aggregate data.
