# Observability and SLOs

This document outlines the operational visibility and Service Level Objectives (SLOs) required to run SALESTORM reliably.

## Metrics
Critical metrics exposed for Prometheus/Grafana monitoring:
- Request rate (throughput)
- Request latency (p50, p95, p99)
- Error rate (HTTP 4xx, 5xx)
- Admission acceptance rate vs. Admission rejection rate
- Inventory reservation success rate vs. failure rate (oversold/empty)
- Payment success rate vs. failure rate
- Order creation success rate
- Order recovery count (asynchronous fallbacks)
- Event backlog (queue depth)
- DLQ size
- Database latency & active connections
- Redis latency
- Circuit breaker state (OPEN/CLOSED/HALF_OPEN)
- Queue processing latency

## Logs
All services utilize structured JSON logs containing:
- `timestamp`
- `requestId` (for the specific HTTP call)
- `correlationId` (for tracing the entire business workflow across services)
- `service` (e.g., `payment-service`)
- `operation` (e.g., `process_payment`)
- `status` (success, failure, warning)
- `error code`
- `latency` (ms)

## Distributed Tracing
Tracing maps the full purchase journey to identify bottlenecks:
`Customer → API Gateway → Admission Control → Reservation → Checkout → Payment → Order → Fulfilment`

## Alerts
PagerDuty/Opsgenie alerts trigger on:
- High error rate (e.g., > 1% 5xx errors)
- High latency (e.g., p95 > 200ms)
- Inventory reservation failures (unexpected database errors)
- Payment failure spike
- Order recovery backlog (queue not draining)
- DLQ growth (poison messages detected)
- Database saturation (connection pool exhausted)
- Redis failure
- Circuit breaker stuck in OPEN state

## Service Level Objectives (SLOs) & Targets

**It is crucial to distinguish strict business invariants (e.g., no overselling) from performance targets.**

- **Latency Target:** $<10\text{ ms}$ for ingress admission.
- **Reality Check:** *The current prototype simulation observed approximately 62.6 ms average ingress latency, so the <10 ms figure remains a target rather than an achieved result.* We do not hide this limitation. Optimization is required to meet the strict latency SLO.

## Prototype Validation Reference
*(The following are prototype validation results from local simulation, NOT production benchmarks).*
- 10,000 concurrent requests simulated.
- 100 successful unique reservations.
- 0 oversold units.
- 0 duplicate orders.
- 94 payment successes, 6 payment failures.
- 94 orders recovered asynchronously after simulated Order Service outage.
- 6 units remaining available (correctly released from failed payments).
- 14 automated tests passed.
- Observed average ingress latency: 62.6 ms.
