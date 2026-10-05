# Traffic Admission and Load Shedding

*This is a CRITICAL SALESTORM document detailing how the system survives extreme traffic spikes.*

## Why Admission Control is Mandatory
Sending 10,000+ concurrent requests directly into the inventory, payment, or order services is unsafe. If 10,000 requests concurrently attempt to update the available quantity of a single row in PostgreSQL, the resulting lock contention will exhaust connection pools, spike CPU, and bring down the entire database, causing a complete system outage.

## The Strategy

```text
Traffic Burst
    |
    v
Admission Controller (e.g., Redis Token Bucket)
    |
    +---- Accepted ----> Purchase Flow (Transactional Services)
    |
    +---- Rejected ----> Fast Failure / HTTP 429
```

## 1. Admission Control & Rate Limiting
The system enforces strict limits on how many requests can proceed to the transactional layer per second. This is typically implemented via a **Token bucket** or equivalent algorithm stored in low-latency infrastructure like Redis.

## 2. Load Shedding & HTTP 429
When the token bucket is empty, the Admission Controller immediately sheds the load. It returns a fast **HTTP 429 Too Many Requests** to the client. This happens in memory or via Redis, costing almost zero CPU compared to a database transaction.

## 3. Fast Rejection
Failing fast is a core reliability principle. Rather than queuing requests indefinitely and causing client-side timeouts (which lead to more retries), the system explicitly tells the client the system is busy.

## 4. Backpressure
If internal queues fill up, services signal backpressure up the chain, dynamically tightening the admission control limits.

## 5. Protection of Transactional Capacity
The admission controller protects the scarce transactional resources (database connections and row locks). By only allowing a manageable trickle of traffic through (e.g., 200 req/s out of 10,000 req/s), the database remains healthy and responsive.

## 6. Fairness & Prioritization
Advanced implementations may prioritize requests based on user tiers, though standard flash sales often rely on strict FIFO fairness.

## Important Distinctions
- **Correctness vs. Admission:** Do not claim admission control alone guarantees inventory correctness. Even if traffic is throttled to 10 req/s, concurrent requests can still oversell if database concurrency isn't handled properly. Inventory correctness still depends entirely on transactional concurrency control.
- **Volume vs. Duplicates:** Admission control handles *volume*. Duplicates (the same user clicking "Buy" twice) are handled separately via Idempotency Keys.
