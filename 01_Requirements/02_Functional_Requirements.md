# SALESTORM - Functional Requirements

**FR-001 Customer and Product Discovery**
- **Requirement:** The system MUST allow querying of product inventory.
- **Description:** Exposes read-only endpoints to check current product availability.
- **Expected behavior:** Returns current available stock.
- **Failure/edge case:** Cached reads may be slightly stale, but reservation attempts rely on the authoritative database.
- **Priority:** Medium

**FR-002 Cart Management**
- **Requirement:** Support single-item checkout flow.
- **Description:** For the flash sale, the system processes direct buy requests rather than complex multi-item carts.
- **Expected behavior:** A buy request specifies user, product, and quantity.
- **Failure/edge case:** Invalid quantities are rejected.
- **Priority:** Low (Simplified for flash-sale context).

**FR-003 Flash Sale / Deal Management**
- **Requirement:** The system MUST support configuring a flash sale event.
- **Description:** The inventory is seeded with a specific initial quantity prior to the sale.
- **Expected behavior:** Inventory is initialized securely before traffic arrives.
- **Failure/edge case:** Modifying inventory during the sale requires careful concurrency control.
- **Priority:** High

**FR-004 Inventory Availability**
- **Requirement:** The system MUST accurately track available, reserved, and sold inventory.
- **Description:** Total units = Available + Reserved + Sold.
- **Expected behavior:** Sum of states always equals initial stock.
- **Failure/edge case:** Concurrency anomalies must not violate this equation.
- **Priority:** High

**FR-005 Inventory Reservation**
- **Requirement:** The system MUST atomically reserve inventory.
- **Description:** Transitions stock from available to reserved.
- **Expected behavior:** Reservation succeeds if stock is available; fails if depleted.
- **Failure/edge case:** Concurrent requests for the last item must result in only one success.
- **Priority:** High

**FR-006 Reservation Expiry**
- **Requirement:** The system MUST enforce a Time-To-Live (TTL) on reservations.
- **Description:** Unpaid reservations expire after a set duration.
- **Expected behavior:** Expired reservations are marked as timed out.
- **Failure/edge case:** Clock skew or delayed processing must not allow processing of expired reservations.
- **Priority:** High

**FR-007 Reservation Release**
- **Requirement:** The system MUST release inventory from expired or failed reservations.
- **Description:** Compensating transactions return reserved stock to the available pool.
- **Expected behavior:** Available stock increments when a reservation is released.
- **Failure/edge case:** Release logic must be idempotent to prevent double-incrementing.
- **Priority:** High

**FR-008 Checkout**
- **Requirement:** The system MUST orchestrate the checkout flow.
- **Description:** Validates reservation, initiates payment, and transitions to order creation.
- **Expected behavior:** End-to-end flow executes reliably.
- **Failure/edge case:** Any step failure must gracefully abort or recover.
- **Priority:** High

**FR-009 Payment Processing**
- **Requirement:** The system MUST integrate with a simulated payment provider.
- **Description:** Submits payment requests and handles success, failure, or timeout responses.
- **Expected behavior:** Successful payments proceed to order creation; failures trigger reservation release.
- **Failure/edge case:** Provider timeouts require polling or eventual consistency mechanisms.
- **Priority:** High

**FR-010 Payment Idempotency**
- **Requirement:** Payment processing MUST be idempotent.
- **Description:** Repeated payment requests for the same reservation must not double-charge.
- **Expected behavior:** Second attempt returns the outcome of the first attempt.
- **Failure/edge case:** Concurrent identical payment requests must be serialized.
- **Priority:** High

**FR-011 Order Creation**
- **Requirement:** The system MUST create an order upon successful payment.
- **Description:** Records the final confirmed state of the transaction.
- **Expected behavior:** Order is durably persisted.
- **Failure/edge case:** Order creation failure must be recoverable (e.g., via events).
- **Priority:** High

**FR-012 Order State Management**
- **Requirement:** The system MUST track order states (e.g., Pending, Confirmed).
- **Description:** Maintains the lifecycle of an order.
- **Expected behavior:** State transitions follow a defined state machine.
- **Failure/edge case:** Invalid state transitions are rejected.
- **Priority:** Medium

**FR-013 Fulfilment / Shipment**
- **Requirement:** Out of scope for this resilience prototype.
- **Priority:** None

**FR-014 Notifications**
- **Requirement:** Out of scope for this resilience prototype.
- **Priority:** None

**FR-015 Duplicate Request Handling**
- **Requirement:** The system MUST handle duplicate ingress requests safely.
- **Description:** Uses Idempotency-Key headers to identify duplicates.
- **Expected behavior:** Duplicate requests return the original response without side effects.
- **Failure/edge case:** Fast-path cache misses must still be caught by database constraints.
- **Priority:** High

**FR-016 Failure Recovery**
- **Requirement:** The system MUST recover from partial component outages.
- **Description:** e.g., if the Order Service goes offline.
- **Expected behavior:** Transactions are buffered or logged and retried upon recovery.
- **Failure/edge case:** Persistent failures may require manual operator intervention (dead-lettering).
- **Priority:** High

**FR-017 Asynchronous Event Processing**
- **Requirement:** The system MUST use events to decouple domains.
- **Description:** Payment success emits an event consumed by the Order Service.
- **Expected behavior:** Events are processed reliably at least once.
- **Failure/edge case:** Duplicate event delivery must be handled idempotently.
- **Priority:** High

**FR-018 Dead-Letter / Failed Event Handling**
- **Requirement:** The system MUST isolate unprocessable events.
- **Description:** Events failing multiple retries are moved to a dead-letter queue.
- **Expected behavior:** System continues processing healthy events while isolating bad ones.
- **Failure/edge case:** DLQ overflow requires monitoring.
- **Priority:** Medium

**FR-019 Observability**
- **Requirement:** The system MUST expose telemetry data.
- **Description:** Metrics include request volume, latency, queue depth, and inventory levels.
- **Expected behavior:** Data is available for real-time dashboards.
- **Failure/edge case:** Telemetry failure must not impact core transaction processing.
- **Priority:** High

**FR-020 Security Controls**
- **Requirement:** The system MUST restrict administrative endpoints.
- **Description:** Operations like resetting inventory require authorization.
- **Expected behavior:** Unauthorized requests are rejected.
- **Failure/edge case:** Misconfiguration could expose admin controls.
- **Priority:** Medium
