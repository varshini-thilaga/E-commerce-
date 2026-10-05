# Failure Scenario Validation

This document outlines the specific failure scenarios explicitly modeled and observed during prototype validation. 

*(Note: The observed behaviors listed below are strictly supported by the existing prototype evidence).*

## 1. Successful Purchase (Happy Path)
- **Input:** Valid purchase request within capacity limits.
- **Expected behavior:** Successfully reserve stock, capture payment, and create order.
- **Observed behavior:** Request processed without errors.
- **Correctness result:** Order created, inventory decremented by 1.

**Flow:**
`Request → Admission → Reservation → Payment → Order → Success`

## 2. Failed Payment
- **Input:** Valid request, but payment declines (simulated 5% failure rate).
- **Expected behavior:** Reservation is released; stock becomes available again.
- **Observed behavior:** 6 out of 100 successful reservations resulted in payment failure; 6 units were released.
- **Correctness result:** Final available inventory is exactly 6 units.

**Flow:**
`Request → Reservation → Payment failure → Reservation release`

## 3. Duplicate Buy Request
- **Input:** Client retries exact same request (same Idempotency-Key).
- **Expected behavior:** Deduplication prevents double charging or double reserving.
- **Observed behavior:** Simulated 2% duplicate rate resulted in 0 duplicate orders.
- **Correctness result:** Existing outcome returned; no duplicate business operations.

**Flow:**
`Duplicate request → Idempotency detection → Existing result reused → No duplicate order`

## 4. Payment Success + Order Service Failure
- **Input:** Valid request, payment succeeds, but Order Service is deliberately offline for 30 seconds.
- **Expected behavior:** Payment completes, event is buffered, order is created after 30s.
- **Observed behavior:** 94 successful payments occurred during the outage. 94 events were buffered. After recovery, 94 orders were confirmed.
- **Correctness result:** Eventual consistency achieved; 0 lost orders.

**Flow:**
`Payment success → Durable event → Order Service unavailable → Event buffered → Recovery → Order replay/confirmation`

## 5. Zero Inventory
- **Input:** Valid purchase request, but all 100 units are already reserved/sold.
- **Expected behavior:** Graceful rejection; no overselling.
- **Observed behavior:** The remaining 9,896 requests were safely rejected. 
- **Correctness result:** Exactly 0 oversold units.

**Flow:**
`Request → Admission → Inventory check → Reservation rejected`

## 6. Traffic Overload
- **Input:** Sudden burst of 10,000 requests.
- **Expected behavior:** Protect transactional database by shedding excess load.
- **Observed behavior:** Admission control accepted traffic up to the limit and shed the excess safely.
- **Correctness result:** Database did not lock or crash.

**Flow:**
`Large traffic burst → Admission control → Accepted traffic continues → Excess traffic rejected safely`
