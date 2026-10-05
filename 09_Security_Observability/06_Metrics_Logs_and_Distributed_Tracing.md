# Metrics, Logs, and Distributed Tracing

This document defines the specific data collected to defend and monitor SALESTORM.

## 1. Metrics

**Traffic:**
- `requests/sec`: Total ingress volume.
- `concurrent requests`: Active connections.
- `accepted requests`: Traffic passing admission control.
- `rejected requests`: Traffic blocked by admission control.
- `HTTP 429 rate`: Load shedding frequency.

**Latency:**
- `p50`, `p95`, `p99`: Percentile response times.
- `endpoint latency`: Latency segmented by route.

**Inventory:**
- `reservation attempts`: Intent to purchase.
- `reservation success`: Stock successfully allocated.
- `reservation failure`: Rejected due to 0 stock.
- `stock remaining`: Current available inventory gauge.
- `reservation expiry`: Timeouts.
- `reservation release`: Stock returned.

**Payment:**
- `payment attempts`
- `payment success`
- `payment failure`
- `payment timeout`
- `duplicate payment attempts`: Caught by idempotency checks.

**Orders:**
- `orders created`
- `order creation failure`
- `order recovery count`
- `pending recovery count`: Orders stuck awaiting downstream recovery.

**Messaging:**
- `event throughput`
- `consumer lag`: Delay in processing events.
- `retry count`
- `DLQ size`: Dead letters requiring manual intervention.

**Infrastructure:**
- `DB latency`, `DB connection utilization`
- `Redis latency`, `Redis errors`
- `service health`: Up/down status.

**Security:**
- `authentication failures`, `authorization failures`
- `suspicious request volume`
- `rate-limit violations`

## 2. Logging

Application logs use a structured JSON format to enable rapid querying.

**Recommended Fields:**
- `timestamp`
- `level` (INFO, WARN, ERROR)
- `service`
- `requestId` (Specific to the edge request)
- `correlationId` (Spans the entire business transaction)
- `traceId`
- `userId` (Where appropriate)
- `operation`
- `resourceId`
- `status`
- `errorCode`
- `latencyMs`

## 3. Distributed Tracing

Tracing tracks the entire lifecycle of a purchase journey across decouple domains.

**Trace Path:**
`Customer → API Gateway → Admission Controller → Checkout → Reservation → Payment → Order → Notification`

**Mechanism:** 
The API Gateway generates a `correlationId` and passes it via HTTP headers (e.g., `X-Correlation-ID`) to downstream services, and inside event envelopes. This allows operators to query the observability platform for a single ID and see every system involved in that specific user's purchase.

*Rule: Do not expose secrets or sensitive payment data in traces.*
