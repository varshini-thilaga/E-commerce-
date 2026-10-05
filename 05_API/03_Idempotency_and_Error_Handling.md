# SALESTORM - Idempotency & Error Handling

This is a CRITICAL SALESTORM document defining how the system safely processes duplicate requests and standardizes error responses.

## 1. Idempotency

Idempotency guarantees that a given operation produces the exact same result no matter how many times it is executed. In a high-concurrency flash sale, clients will aggressively retry requests due to network timeouts. 

Idempotency MUST be enforced for the following critical operations:
- Reservation
- Checkout
- Payment
- Order creation
- Event/message consumption

### The `Idempotency-Key` Header

Clients must generate a unique identifier (UUID) and pass it in the request header.
**Example:** `Idempotency-Key: 8b7f-example-key`

### Expected Behavior

**1. First request:**
- The system processes the business operation.
- The system persists the `Idempotency-Key` and the resulting response payload (the idempotency record) durably.

**2. Repeated request with same key and same business parameters:**
- The system detects the key.
- The system *does not* execute the business logic or create another business operation (e.g., does not decrement inventory again).
- The system returns the previously established result.

**3. Same key with different business parameters:**
- If a client reuses an idempotency key but alters the payload (e.g., changing quantity from 1 to 2), the system MUST reject it as an idempotency conflict.

**4. Concurrent duplicate requests:**
- If multiple requests with the identical key arrive at the exact same millisecond, database locks or unique constraints guarantee that only one request "wins" and executes the business operation.
- The "losing" duplicate requests must safely block or retry until the winner completes, then observe and return the same established outcome.

### Idempotency Storage
Idempotency storage must be durable enough for the required retry window (e.g., 24-48 hours). Caching layers (like Redis) can provide fast-path deduplication, but authoritative deduplication is backed by the relational database.

---

## 2. Payment Idempotency Flow

Payment processing is highly susceptible to network unreliability. The flow is as follows:

```text
Client
  |
  v
Payment API
  |
  v
Idempotency Check
  |
  +---- Existing result ---> Return existing result
  |
  +---- New request -------> Process payment with External Provider
                              |
                              v
                         Persist result locally
```

**CRITICAL NOTE:** Local database state alone does not automatically make an external payment provider call atomic with local persistence. If the network drops *after* the external provider charges the card but *before* the system persists the local result, the outcome is unknown.
**Recovery/Reconciliation:** The system must implement webhook listeners or scheduled polling to reconcile unknown payment states with the provider, avoiding double charges.

---

## 3. Error Handling

Error responses follow a consistent, structured JSON design. **Never expose stack traces, SQL errors, secrets, tokens, or internal implementation details in API responses.**

### Standard Error Response Format
```json
{
  "error": {
    "code": "INSUFFICIENT_INVENTORY",
    "message": "Requested quantity is not available",
    "requestId": "req-12345",
    "details": {}
  }
}
```
*The `requestId` serves as a correlation ID for tracing the request across microservices in centralized logging.*

### Examples of Error Codes

- **Validation failure:**
  - `code`: `VALIDATION_ERROR` (HTTP 400)
  - `message`: "Invalid request payload format."
- **Authentication failure:**
  - `code`: `UNAUTHORIZED` (HTTP 401)
  - `message`: "Missing or invalid Bearer token."
- **Authorization failure:**
  - `code`: `FORBIDDEN` (HTTP 403)
  - `message`: "Insufficient permissions."
- **Product not found:**
  - `code`: `PRODUCT_NOT_FOUND` (HTTP 404)
  - `message`: "The requested product does not exist."
- **Insufficient inventory:**
  - `code`: `INSUFFICIENT_INVENTORY` (HTTP 409)
  - `message`: "Requested quantity is not available."
- **Reservation expired:**
  - `code`: `RESERVATION_EXPIRED` (HTTP 409)
  - `message`: "The reservation has timed out."
- **Duplicate/idempotency conflict:**
  - `code`: `IDEMPOTENCY_CONFLICT` (HTTP 409)
  - `message`: "Idempotency key reused with different payload."
- **Payment failure:**
  - `code`: `PAYMENT_DECLINED` (HTTP 402)
  - `message`: "The payment method was declined by the provider."
- **Payment timeout:**
  - `code`: `PAYMENT_TIMEOUT` (HTTP 504)
  - `message`: "The payment provider did not respond in time."
- **Order service unavailable:**
  - `code`: `SERVICE_UNAVAILABLE` (HTTP 503)
  - `message`: "The order service is temporarily offline."
- **Rate limit rejection:**
  - `code`: `TOO_MANY_REQUESTS` (HTTP 429)
  - `message`: "Ingress capacity exceeded. Please try again."
- **Internal server error:**
  - `code`: `INTERNAL_ERROR` (HTTP 500)
  - `message`: "An unexpected system error occurred."
