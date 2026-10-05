# Single Responsibility Principle

A class/module should have one clear responsibility or reason to change. In the context of the SALESTORM platform, SRP is critical for isolating the intense performance demands of flash sales from the strict consistency requirements of inventory.

## Application to SALESTORM

We apply SRP by strictly separating business domains.

### ReservationService
- **Responsibility:** Coordinates reservation business rules and atomic inventory decrement.
- **Constraint:** Does not directly become responsible for payment processing or admission rate-limiting.

### PaymentService
- **Responsibility:** Coordinates payment business logic and tracks payment intent state.
- **Constraint:** Does not own inventory mutation or order lifecycle transitions.

### OrderService
- **Responsibility:** Owns the order lifecycle and eventual consistency of fulfillment.
- **Constraint:** Does not directly implement payment-provider details.

### IdempotencyService *(Proposed architectural component)*
- **Responsibility:** Handles duplicate-request filtering and idempotency record storage.
- **Constraint:** Does not contain business logic for orders or payments.

### OrderStateMachine *(LLD design abstraction)*
- **Responsibility:** Owns and validates valid order state transitions.
- **Constraint:** Does not handle database persistence or event publishing.

### RecoveryHandler *(LLD design abstraction)*
- **Responsibility:** Processes dead-letter queues and replays events.
- **Constraint:** Does not execute synchronous HTTP API requests.

## The Benefit
A change to payment logic (e.g., adding 3D Secure verification) should not require changing the atomic SQL inventory logic. A change to order-state rules should not require changing the payment-provider integration.

### Conceptual Example

**Before (Violates SRP):**
```text
Large CheckoutService
├── inventory logic
├── payment logic
├── order logic
├── notification logic
└── recovery logic
```
*If a bug in notification logic crashes the process, the inventory transaction might roll back or hang.*

**After (Follows SRP):**
```text
CheckoutService (Orchestrator only)
ReservationService
PaymentService
OrderService
NotificationService
RecoveryHandler
```
**(Note: These are conceptual LLD abstractions defining boundaries, verifying that the actual prototype separates these concerns logically.)**
