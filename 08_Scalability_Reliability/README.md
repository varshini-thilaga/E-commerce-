# SALESTORM Scalability & Reliability

## 1. Purpose
This folder details the architectural strategies used by the STORMSHIELD Flash-Sale Resilience Platform to guarantee strict business invariants under extreme load, handle component failures gracefully, and maintain operational visibility.

## 2. Reliability Goals
The primary goal is strict protection of business constraints (no overselling, idempotent operations) even when external dependencies fail, network partitions occur, or downstream services crash.

## 3. Scalability Goals
The system must dynamically accommodate massive traffic spikes without exhausting scarce transactional database resources.

## 4. Workload Assumptions
The architecture is designed to reason about:
- **Normal workload:** ~10,000 req/s.
- **Flash-sale workload:** Spikes approaching ~500,000 req/s.
*(Note: These are architectural workload assumptions to guide design choices, not measured production capacity benchmarks of the current local prototype).*

## 5. Critical Invariants
The system is built around mathematically preventing `available_quantity < 0` and ensuring duplicate Idempotency-Keys do not trigger duplicate business operations.

## 6. Traffic Management
Aggressive admission control blocks excessive traffic from ever reaching transactional bottlenecks.

## 7. Inventory Consistency
PostgreSQL acts as the transactional source of truth to guarantee ACID compliance for inventory decrements.

## 8. Payment and Order Recovery
Asynchronous event-driven recovery handles the inevitable failures of external payment providers or downstream Order services.

## 9. Caching and Database Scaling
Redis is used for fast-path admission, while database read-replicas scale reads. (Sharding remains a future option).

## 10. Event Processing
Message queues with Dead Letter Queues (DLQ) support at-least-once delivery to idempotent consumers.

## 11. Observability
Structured logging, distributed tracing, and critical metrics allow operators to monitor circuit breakers and DLQ depth.

## 12. Failure Scenarios
The system is validated against payment timeouts, payment failures, duplicate requests, and total Order Service outages.

---

## 13. Prototype Validation Reference
The current local prototype successfully executed the following challenge simulation:
- **10,000** concurrent requests burst
- **100** available units
- **100** successful unique reservations
- **0** oversold units
- **0** duplicate orders
- **94** payment successes, **6** payment failures
- **94** orders successfully recovered after a 30-second Order Service outage
- **6** units gracefully returned to available stock
- **14** automated tests passed

*These are prototype validation results proving the architectural correctness. They are NOT production benchmarks.*

---

## 14. Strict Guarantees vs Targets

**STRICT GUARANTEES:**
- Inventory must never be oversold.
- Duplicate business operations must be prevented for the same idempotency key.
- Reservations can expire and release correctly.
- Successful payment must have a recovery path if Order Service fails.

**TARGETS / DESIGN GOALS:**
- <10 ms latency target.
- ~10k req/s normal workload.
- Reasoning about ~500k req/s flash-sale workload.

**Important Note:** Do not present targets as achieved measurements. The current prototype simulation observed approximately **62.6 ms** average ingress latency, so the <10 ms figure remains an optimization target rather than an achieved result.

## 15. Key Trade-offs
We accept higher latency on the recovery path to ensure strict correctness. We reject the complexity of exactly-once message delivery in favor of at-least-once delivery with strictly idempotent consumers.
