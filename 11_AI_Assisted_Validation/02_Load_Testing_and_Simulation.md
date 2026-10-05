# Load Testing and Simulation

This document outlines the results of the existing prototype simulation designed to validate concurrency and resilience behavior.

### Workload
- **10,000** concurrent requests

### Inventory
- **100** units available

### Observed Results

- **100** successful unique reservations
- **9,896** safely rejected (via admission control and strict 0-stock boundaries)
- **0** oversold units
- **0** duplicate orders
- **94** payment successes
- **6** payment failures
- **94** events buffered during Order Service outage
- **94** orders replayed/confirmed after outage recovery
- **6** units remaining available
- **Average ingress latency:** 62.6 ms

## Important Distinction: Concurrency vs. Throughput

It is critical to distinguish between concurrent load and sustained throughput capacity. 

**10,000 concurrent requests is NOT the same as 10,000 requests per second.** 

The simulation script fires 10,000 requests simultaneously to generate a massive burst of contention against a single database row, testing lock behavior, conditional updates, and admission control thresholds. 

This simulation validates concurrency behavior and correctness under pressure rather than proving production throughput capacity. We do not convert this result into an unsupported throughput claim (e.g., we do not claim the prototype achieved 500k req/s). Similarly, the observed average ingress latency was 62.6 ms, indicating that the target of <10 ms ingress latency remains a design goal rather than an achieved benchmark.
