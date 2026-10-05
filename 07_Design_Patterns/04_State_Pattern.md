# State Pattern - Order and Reservation Lifecycle

*This is a CRITICAL SALESTORM pattern for managing the complex lifecycle of flash-sale inventory and asynchronous orders.*

## Problem
Order and Reservation behaviors depend entirely on their current state. For example, a reservation can only be confirmed if it is currently in the `RESERVED` state. Attempting to confirm a reservation that has already `EXPIRED` is a critical business error. Managing these transitions with massive `if/else` or `switch` blocks leads to brittle, error-prone logic.

## Defined States

### Reservation States:
- `CREATED`
- `RESERVED`
- `CONFIRMED`
- `RELEASED`
- `EXPIRED`

### Order States:
- `PENDING`
- `PAYMENT_PENDING`
- `PAYMENT_CONFIRMED`
- `ORDER_CREATED`
- `FULFILMENT_PENDING`
- `SHIPPED`
- `DELIVERED`
- `PAYMENT_FAILED`
- `CANCELLED`
- `RECOVERY_PENDING`

## Structure (Conceptual LLD Design Abstraction)

```text
Order
   |
   v
OrderStateMachine
   |
   +---- PendingState
   +---- PaymentPendingState
   +---- PaymentConfirmedState
   +---- OrderCreatedState
   +---- RecoveryPendingState
   +---- CancelledState
   ...
```

## Explanation

The `OrderStateMachine` explicitly defines:
- **Valid transitions:** e.g., `PAYMENT_PENDING` -> `PAYMENT_CONFIRMED`.
- **Invalid transitions:** e.g., Transitioning from `CANCELLED` to `DELIVERED` must throw a state exception.
- **State-specific behavior:** Actionable logic tied to a state (e.g., entering `EXPIRED` triggers inventory release).
- **Recovery transitions:** Safe paths for asynchronous event replay (e.g., `RECOVERY_PENDING` -> `ORDER_CREATED`).

## Crucial Trade-off

While the formal Gang of Four "State Pattern" suggests creating a distinct polymorphic class for every individual state (e.g., `class PendingState implements OrderState`), this is often overkill. 

**If the state logic remains small and transitions are strictly linear, a simple Enum accompanied by a transition table (or a lightweight finite state machine library) is highly preferable** to a massive, fragmented class hierarchy. A full State-pattern class hierarchy should only be implemented if each state requires vastly different, complex behavioral logic. *In the SALESTORM prototype, a lightweight approach is utilized.*
