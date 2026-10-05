# SALESTORM - API Specification

The following endpoints define the core capabilities of the STORMSHIELD platform. 

*Note: Endpoints marked as [Prototype API] are implemented in the current validation prototype. Endpoints marked as [Proposed API] represent the full envisioned architecture.*

---

## PRODUCT

### GET /api/v1/products [Proposed API]
- **Purpose:** List available products and flash sales.
- **Authentication:** Optional
- **Required headers:** None
- **Path/query parameters:** `?page=1&limit=20`
- **Request body:** None
- **Successful response:** `200 OK` (List of products)
- **Possible errors:** `500 Internal Server Error`
- **Idempotency requirement:** Not required (Read-only)
- **Owning service:** Product Service

### GET /api/v1/products/{productId} [Proposed API]
- **Purpose:** Get specific product details and live availability.
- **Authentication:** Optional
- **Path parameters:** `productId` (string)
- **Successful response:** `200 OK` (Product object)
- **Possible errors:** `404 Not Found`
- **Idempotency requirement:** Not required
- **Owning service:** Product Service / Inventory Service

---

## CART

### POST /api/v1/carts/{cartId}/items [Proposed API]
- **Purpose:** Add an item to a user's cart.
- **Authentication:** Required (Bearer Token)
- **Path parameters:** `cartId` (string)
- **Request body:** `{"productId": "P100", "quantity": 1}`
- **Successful response:** `200 OK`
- **Possible errors:** `400 Bad Request`, `401 Unauthorized`, `404 Not Found`
- **Idempotency requirement:** Recommended
- **Owning service:** Cart Service

### GET /api/v1/carts/{cartId} [Proposed API]
- **Purpose:** Retrieve the current cart state.
- **Authentication:** Required
- **Successful response:** `200 OK`
- **Possible errors:** `401 Unauthorized`, `404 Not Found`
- **Idempotency requirement:** Not required
- **Owning service:** Cart Service

---

## RESERVATION

### POST /api/v1/reservations [Prototype API]
- **Purpose:** Atomically reserve inventory for a flash-sale item.
- **Authentication:** Required
- **Required headers:** `Idempotency-Key: <unique-request-key>`
- **Request body:**
```json
{
  "productId": "P100",
  "customerId": "C100",
  "quantity": 1
}
```
- **Successful response:** `201 Created`
```json
{
  "reservationId": "res-9876",
  "productId": "P100",
  "quantity": 1,
  "status": "RESERVED",
  "expiresAt": "2026-10-05T10:05:00Z"
}
```
- **Possible errors:** 
  - `409 Conflict` (Inventory depleted)
  - `429 Too Many Requests` (Load shed by admission control)
  - `400 Bad Request`
- **Idempotency requirement:** STRICT
- **Owning service:** Inventory Service

### POST /api/v1/reservations/{reservationId}/confirm [Proposed API]
- **Purpose:** Lock a reservation permanently upon successful checkout.
- **Authentication:** Internal / Required
- **Idempotency requirement:** STRICT
- **Owning service:** Inventory Service

### POST /api/v1/reservations/{reservationId}/release [Proposed API]
- **Purpose:** Manually release a reservation (or invoked by expiry worker).
- **Authentication:** Internal
- **Idempotency requirement:** STRICT
- **Owning service:** Inventory Service

---

## CHECKOUT

### POST /api/v1/checkout [Proposed API]
- **Purpose:** Orchestrate the transition from reserved items to payment.
- **Authentication:** Required
- **Required headers:** `Idempotency-Key: <unique-request-key>`
- **Request body:** `{"reservationId": "res-9876", "paymentDetails": "..."}`
- **Successful response:** `202 Accepted`
- **Possible errors:** `400 Bad Request`, `409 Conflict` (Reservation expired)
- **Idempotency requirement:** STRICT
- **Owning service:** Checkout Service

---

## PAYMENT

### POST /api/v1/payments [Prototype API]
- **Purpose:** Process a payment against a reservation.
- **Authentication:** Required
- **Required headers:** `Idempotency-Key: <unique-request-key>`
- **Request body:**
```json
{
  "orderId": "ord-123",
  "amount": 99.99,
  "paymentMethodReference": "pm_tok_xyz123"
}
```
*(Note: Raw payment credentials are NEVER transmitted in this payload; tokens are used instead.)*
- **Successful response:** `200 OK` (Payment successful) or `202 Accepted` (Async processing)
- **Possible errors:** `400 Bad Request`, `402 Payment Required` (Declined), `503 Service Unavailable`, `504 Gateway Timeout`
- **Idempotency requirement:** STRICT
- **Owning service:** Payment Service

### GET /api/v1/payments/{paymentId} [Proposed API]
- **Purpose:** Check the status of a payment.
- **Successful response:** `200 OK`
- **Owning service:** Payment Service

---

## ORDER

### POST /api/v1/orders [Proposed API]
- **Purpose:** Create a confirmed order record (typically invoked via internal event).
- **Authentication:** Internal Service Token
- **Required headers:** `Idempotency-Key: <unique-request-key>`
- **Idempotency requirement:** STRICT
- **Owning service:** Order Service

### GET /api/v1/orders/{orderId} [Proposed API]
- **Purpose:** Fetch order details for the customer.
- **Authentication:** Required
- **Successful response:** `200 OK`
- **Possible errors:** `401 Unauthorized`, `404 Not Found`
- **Owning service:** Order Service

---

## SHIPMENT

### GET /api/v1/shipments/{shipmentId} [Proposed API]
- **Purpose:** Track an order's physical shipment.
- **Authentication:** Required
- **Successful response:** `200 OK`
- **Owning service:** Shipment Service
