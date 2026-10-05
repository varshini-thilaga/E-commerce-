# SALESTORM Architecture Decision Records

## Purpose

These Architecture Decision Records (ADRs) document the major architectural choices and trade-offs behind the STORMSHIELD Flash-Sale Resilience Platform. They capture *why* specific choices were made, complementing the requirements and designs found in previous folders.

| ADR | Decision | Main Reason |
|---|---|---|
| ADR-001 | PostgreSQL for transactional source of truth | Strong consistency and relational integrity |
| ADR-002 | Hybrid sync/async | Balance responsiveness and resilience |
| ADR-003 | Atomic conditional inventory update | Prevent overselling under concurrency |
| ADR-004 | Redis for fast-path/cache | Reduce read pressure without owning transactional truth |
| ADR-005 | Transactional reservation consistency | Protect inventory invariant |
| ADR-006 | Durable payment/order recovery | Recover from Order Service failure |
| ADR-007 | Logical service boundaries with pragmatic prototype deployment | Scalability without unnecessary prototype complexity |

## Decision Principles

1. **Correctness before throughput.** A mathematically correct system that sheds load is better than a fast system that oversells.
2. **Protect inventory as the scarce resource.** Do not let external bursts overwhelm the database.
3. **Keep transactional truth in PostgreSQL.** Rely on ACID compliance for financial and inventory states.
4. **Use Redis for acceleration, not correctness.** Fast-path admission control is appropriate; authoritative inventory is not.
5. **Use asynchronous processing where it improves resilience.** Decouple domains to prevent cascading failures.
6. **Make retries bounded and operations idempotent.** Prevent retry storms and duplicate business actions.
7. **Prefer explicit trade-offs over pretending there is a perfect architecture.** Every abstraction has a cost.
8. **Keep the prototype simpler than the production deployment model when appropriate.** Differentiate between logical architecture and hackathon execution.

## Relationship to Other Folders

These ADRs explain the foundational logic for the designs detailed in:
- `01_Requirements`
- `02_HLD`
- `03_LLD`
- `04_Database`
- `05_API`
- `06_SOLID`
- `07_Design_Patterns`
- `08_Scalability_Reliability`
- `09_Security_Observability`

## Prototype Validation Context

Decisions were tested against the following validation constraints:
- 10,000 concurrent requests over 100 available units.
- 0 oversold units in prototype simulation.
- 0 duplicate orders.
- 94 payment successes, 6 payment failures.
- 94 orders recovered asynchronously after Order Service outage.
- 14/14 automated tests passed.
- Observed average ingress latency: 62.6 ms.

*(Note: These are validation outcomes of the architectural concepts, not a production benchmarking claim. The <10 ms latency metric remains a design target.)*
