# Prototype Validation Strategy

This document outlines the validation strategy utilized to verify the SALESTORM architectural constraints.

## Scenario

The prototype was subjected to a simulated flash-sale challenge workload:
- **10,000** concurrent purchase requests
- **Available inventory:** 100 units
- **Payment model:** 95% success rate, 5% failure rate
- **Duplicate request scenario:** 2% of traffic consists of retry bursts using identical Idempotency Keys.
- **Order Service outage:** A deliberate 30-second downstream outage was injected during peak load.

## What the Prototype Attempts to Validate

The prototype serves as an executable proof-of-concept for the following critical architectural components:
- **Traffic admission:** Shedding excess load to protect the database.
- **Inventory concurrency:** Handling extreme row contention gracefully via conditional atomic updates.
- **No overselling:** Mathematically preventing negative inventory.
- **Idempotency:** Safely ignoring duplicate requests without executing duplicate business operations.
- **Payment failure compensation:** Releasing inventory when a payment declines.
- **Order recovery:** Buffering events and replaying them to ensure eventual fulfillment.
- **State transitions:** Moving reservations and orders through a strict lifecycle.
- **Redis integration:** Using Redis for fast-path token-bucket admission control.

## Expected Correctness Properties

The validation is considered successful only if all of the following properties hold true after the simulation concludes:
1. Successful reservations <= 100.
2. `available_quantity` never becomes negative.
3. Duplicate requests do not create duplicate orders.
4. Failed payments successfully release inventory back to the available pool.
5. Payment success survives the injected Order Service interruption.
6. Asynchronous recovery eventually creates valid orders for all successful payments.
