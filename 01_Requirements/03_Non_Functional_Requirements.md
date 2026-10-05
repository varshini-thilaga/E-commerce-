# SALESTORM - Non-Functional Requirements

## 1. Performance
The system must handle intense traffic bursts with minimal latency for admission control. 

## 2. Scalability
The system architecture must support horizontal scaling of edge and application layers.

## 3. Consistency
The inventory balance must be strictly consistent, while downstream processes (like orders) may be eventually consistent.

## 4. Reliability
The system must predictably process successful and failed operations without data loss.

## 5. Availability
The ingress admission layer must remain available even if backend processing is degraded.

## 6. Fault Tolerance
Component failures (like the Order Service or external Payment Provider) must not cause cascading system failure.

## 7. Security
Administrative endpoints and telemetry data should be protected from unauthorized access.

## 8. Observability
The system must generate structured logs, traces, and metrics to provide real-time operational visibility.

## 9. Maintainability
The codebase must be structured using clean architecture principles and separation of concerns.

## 10. Idempotency
All transactional endpoints must be safely retriable without unintended side effects.

## 11. Recovery
The system must be capable of replaying missed events and restoring state after an outage.

## 12. Operability
The platform must support automated deployment, configuration via environment variables, and health checks.

### Requirements Table

| Category | Requirement | Type |
|----------|-------------|------|
| Consistency | Inventory must never be oversold under any load. | Strict Guarantee |
| Idempotency | Duplicate requests must not result in duplicate transactions. | Strict Guarantee |
| Recovery | Confirmed payments must eventually result in confirmed orders. | Strict Guarantee |
| Recovery | Expired reservations must release inventory back to availability. | Strict Guarantee |
| Performance | Ingress processing latency < 10 ms. | Target |
| Scalability | Sustain 500,000 requests/sec peak admission capacity. | Design Goal |
| Fault Tolerance | Ingress remains responsive during Order Service outage. | Target |
| Observability | Metrics and correlation IDs exposed for all requests. | Target |

---

## Current Prototype Validation Reference

*Note: The following metrics are observed validation results from the current local prototype execution, not hard requirements. They represent the system's baseline behavior under simulation.*

- **Traffic:** 10,000 ingress requests
- **Reservations:** 100 successful reservations
- **Oversold:** 0 oversold units
- **Duplicates:** 0 duplicate orders
- **Payments:** 94 successful payments, 6 payment failures
- **Orders:** 94 recovered orders
- **Final Stock:** 6 final available units
- **Latency:** Average ingress latency observed at 62.6 ms
- **Testing:** 14 automated tests passed

**Optimization Note:** The observed ingress latency of 62.6 ms is above the previously stated target of <10 ms. This remains an optimization area for future iterations. This prototype validation proves the architectural resilience and strict guarantees, but is not a claim of 100% production readiness at the maximum design scale.
