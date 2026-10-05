# SALESTORM - Requirements & Assumptions

## 1. Problem Statement
High-scale flash-sale e-commerce platforms frequently encounter catastrophic failures during massive traffic bursts. Common failure modes include database contention, cascading service degradation, and race conditions resulting in oversold inventory or duplicate transactions. The problem is to design a control plane capable of shielding the transactional inventory boundary under extreme burst traffic.

## 2. Project Objective
Design and implement the STORMSHIELD Flash-Sale Resilience Platform. The system must act as a resilience control plane that reliably processes extreme traffic bursts, protects the authoritative inventory state, and ensures correct business outcomes through idempotent, asynchronous workflows.

## 3. Flash-Sale Scenario
The system must support a highly contended flash-sale scenario characterized by:
- 100 available units of a single product.
- Up to 10,000 concurrent purchase requests targeting the available units within seconds.
- Prevention of overselling under high concurrency.
- Prevention of duplicate reservations, orders, and payments.
- Temporary inventory reservations.
- Reservation expiry and release mechanisms.
- Reliable payment and order processing workflows.
- Order recovery mechanisms in the event of partial system failures.
- Horizontal scalability to accommodate load.
- Comprehensive operability and observability.

## 4. Business Requirements
The platform MUST ensure that business invariants are protected regardless of traffic volume or system component failure. The primary business requirement is to maximize successful transactions up to the limit of available inventory without ever exceeding it or charging customers incorrectly.

## 5. Core System Guarantees
- The inventory balance MUST NEVER fall below zero.
- The total number of confirmed sales plus active reservations MUST NEVER exceed the initial inventory.
- Requests with identical idempotency keys MUST NOT trigger duplicate business operations (reservations, payments, or orders).

## 6. Assumptions
- **Inventory Quantity:** The flash-sale item is strictly limited (e.g., 100 units).
- **Concurrent Traffic:** Traffic exhibits extreme skew, with thousands of users attempting to purchase the same item simultaneously.
- **Payment Behavior:** External payment providers will experience failures (e.g., 5% failure rate due to insufficient funds or timeouts).
- **Order-Service Failures:** Downstream systems, such as the Order Service, may experience temporary outages (e.g., up to 30 seconds).
- **External Payment Dependency:** Payment processing is simulated but acts as a distributed, out-of-process dependency.
- **Reservation Expiry:** Temporary reservations have a defined Time-To-Live (TTL); unpaid reservations expire, and stock is released.
- **Database Consistency:** A relational database provides the foundational ACID guarantees necessary for the authoritative inventory state.
- **Authentication/Security:** Requests are assumed to be pre-authenticated or carry verifiable user identifiers at the ingress layer.
- **Observability:** Telemetry and metrics collection operate out-of-band and do not impede the critical transaction path.

## 7. Scope
The scope includes admission control, idempotent reservation management, simulated payment processing, resilient asynchronous order creation, and failure recovery mechanisms. 

## 8. Out of Scope
The following areas are excluded from the current design scope:
- Frontend user interfaces (excluding the operational dashboard).
- User authentication and identity management systems.
- Physical fulfillment, warehousing, and shipping logistics.
- Complex product catalog management.

## 9. Strict Guarantees vs Targets

**STRICT GUARANTEES:**
- Inventory must never be oversold. The mechanism enforcing this invariant must mathematically prevent overallocation.
- Duplicate business operations must be safely handled through idempotency.
- Expired/unpaid reservations must eventually release inventory.
- Successful purchases must eventually reach a valid order state or a defined recovery path.

**TARGETS / DESIGN GOALS:**
- High throughput.
- Low latency.
- Horizontal scalability.
- Graceful degradation under excessive load.
- Operational visibility.

*(Note: The system is designed to reason about scale up to 500,000 requests/sec, but this is a design target, not a strict guarantee of current hardware performance.)*

## 10. Success Criteria
The system is considered successful if it can sustain the simulated challenge workload (10,000 concurrent requests against 100 units) while upholding all strict guarantees, correctly recovering from injected payment and service failures, and providing real-time observability of the system state.
