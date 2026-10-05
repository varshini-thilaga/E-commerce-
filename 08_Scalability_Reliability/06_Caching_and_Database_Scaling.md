# Caching and Database Scaling

This document clarifies the specific roles of Redis and PostgreSQL in scaling the SALESTORM platform.

## Redis (Caching & Admission)

**Potential Uses:**
- **Admission-control support:** High-speed Token Bucket counters for load shedding.
- **Product/catalog caching:** Serving read-heavy traffic for product details.
- **Hot sale metadata:** Storing configuration like sale start/end times.
- **Idempotency fast lookup:** Providing a low-latency check for duplicate requests (falling back to the DB).
- **Short-lived data:** TTL-based session or cart data.

**Important Constraint:** Do not claim Redis guarantees inventory consistency. Redis is volatile. It acts as an optimization layer, not the system of record.

## PostgreSQL (Transactional Source of Truth)

**Role:**
The definitive source of truth for all transactional, relational business state, including:
- Inventory balances
- Reservations
- Orders
- Payments

## Scaling Strategies

- **Cache-aside strategy:** The application checks Redis first. If a cache miss occurs, it reads from PostgreSQL and populates Redis.
- **Cache invalidation & TTL:** Product data has a short TTL to prevent stale data. Updates to inventory invalidate cache keys.
- **Cache stampede protection:** Mutexes or probabilistic early expiration prevent thousands of requests from simultaneously hitting the database when a cache key expires.
- **Database connection pooling:** PgBouncer ensures the database isn't overwhelmed by idle connections from thousands of scaled application containers.
- **Indexing:** Proper indexes on `idempotency_key`, `product_id`, and `status` fields to ensure fast lookups.
- **Read replicas:** Diverting read-only traffic (like checking historical orders) to replica instances to reserve the primary instance's CPU for writes.

## Write Contention vs Sharding

During a flash sale, 10,000 users attempting to buy the exact same SKU (e.g., row ID 100) will hit the exact same database shard, regardless of how many shards exist. 

**Important:** Do not claim sharding is required for the prototype, nor that database replication automatically solves write contention. Write contention on a single hot row is solved via **admission control** and **atomic conditional updates**, not by adding more database servers. Partitioning/sharding is a future scaling option for broader catalog scale, not the immediate solution for single-item flash-sale contention.
