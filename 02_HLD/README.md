# SALESTORM - High-Level Design

## 1. Purpose

This folder documents the High-Level Design (HLD) of SALESTORM, a high-scale e-commerce flash-sale system designed to handle extreme traffic bursts while maintaining strict inventory consistency.

The HLD describes:

- System context
- Major architectural building blocks
- Container/service responsibilities
- Data and infrastructure dependencies
- Synchronous and asynchronous communication
- Scalability approach
- Reliability and failure handling
- Deployment approach

## 2. Problem Context

SALESTORM models a flash-sale scenario where:

- Up to 10,000 concurrent purchase requests may arrive.
- Only 100 units of the product are available.
- The system must prevent inventory overselling.
- Duplicate purchase/payment requests must not create duplicate business operations.
- Temporary inventory reservations must expire or be released when payment fails or the reservation becomes invalid.
- Payment and order processing must have recovery paths when downstream services fail.
- The architecture must support horizontal scaling and graceful degradation.

## 3. Architecture Overview

The proposed architecture separates the major business capabilities and supporting infrastructure into logical components.

### Main request path

```text
Customer
   |
   v
CDN / WAF
   |
   v
Load Balancer
   |
   v
API Gateway
   |
   v
Admission Control
   |
   v
Product / Cart / Sale
   |
   v
Inventory & Reservation
   |
   v
Checkout
   |
   v
Payment
   |
   v
Order
   |
   v
Fulfilment / Shipment
   |
   v
Notification
```

The inventory reservation boundary is the most important consistency boundary because only a limited quantity of stock is available.

## 4. Major Architectural Components

| Component | Responsibility |
|---|---|
| CDN / WAF | Edge protection and traffic filtering |
| Load Balancer | Distributes traffic across application instances |
| API Gateway | Entry point, routing, validation and authentication |
| Admission Control | Controls flash-sale traffic and sheds excess load |
| Product Service | Product discovery and catalog operations |
| Cart Service | Cart and cart-item management |
| Sale Service | Flash-sale/deal configuration |
| Inventory & Reservation | Atomic stock reservation, confirmation, release and expiry |
| Checkout Service | Coordinates reservation, payment and order operations |
| Payment Service | Payment processing, idempotency, timeout and retry handling |
| Order Service | Order creation and lifecycle management |
| Fulfilment / Shipment | Shipment and fulfilment processing |
| Notification Service | Customer notifications |
| Redis | Hot data, caching and idempotency/admission support |
| PostgreSQL | Transactional business data |
| Durable Event Log | Reliable asynchronous processing and recovery |
| Dead Letter Queue | Failed event isolation |
| Observability | Metrics, logs, tracing and alerts |

## 5. Inventory Consistency

Inventory is the critical concurrency boundary.

The reservation operation must be performed transactionally so that two concurrent requests cannot reserve the same unit.

The core invariant is:

```text
available_quantity >= 0
```

The reservation operation should conceptually perform:

```text
BEGIN TRANSACTION

Validate request

Check available quantity

Atomically reduce available quantity

Create reservation record

COMMIT
```

If sufficient inventory does not exist, the reservation is rejected.

This design ensures that successful reservations cannot exceed the available inventory.

## 6. Admission Control

During a flash sale, the system may receive far more requests than the downstream transactional system can safely process.

Admission control therefore sits early in the request path.

```text
Large Traffic Burst
       |
       v
Admission Control
       |
       +---- Accepted ----> Purchase Flow
       |
       +---- Rejected ----> Fast Failure
```

The purpose is to protect the inventory and transactional layers from uncontrolled request amplification.

## 7. Synchronous Communication

Synchronous communication is used where the caller requires an immediate business result.

Examples include:

- API Gateway to application services
- Checkout to inventory reservation
- Checkout to payment
- Customer-facing purchase confirmation

Synchronous operations should use appropriate:

- Timeouts
- Validation
- Idempotency
- Bounded retries where safe
- Circuit breakers for unstable external dependencies

## 8. Asynchronous Communication

Asynchronous communication is used where work can continue independently or where temporary downstream failure should not cause immediate data loss.

Examples include:

- Order events
- Notifications
- Fulfilment events
- Recovery events
- Failed event processing

A durable event mechanism allows events to remain available while a downstream service is temporarily unavailable.

Failed messages can be isolated through a Dead Letter Queue and processed through a controlled recovery mechanism.

## 9. Payment and Order Reliability

Payment and order processing are separate reliability boundaries.

A successful payment must not be lost simply because the Order Service is temporarily unavailable.

The recovery concept is:

```text
Payment Success
      |
      v
Order Creation
      |
      +---- Order Service Available ----> Order Created
      |
      +---- Order Service Unavailable
                    |
                    v
             Durable Event
                    |
                    v
              Recovery Worker
                    |
                    v
              Order Creation
```

Payment operations must also use idempotency keys so that retrying the same business request does not create duplicate transactions.

## 10. Scalability

The architecture is designed around horizontal scaling.

Potential scaling mechanisms include:

- Multiple API/application instances
- Load balancing
- Stateless request handling
- Redis caching
- Asynchronous workers
- Database read scaling where appropriate
- Admission control
- Traffic shedding

The architecture should reason about normal traffic of approximately 10,000 requests/sec and flash-sale conditions approaching approximately 500,000 requests/sec as a design scenario.

These are architectural workload assumptions, not claims that the current prototype has been benchmarked at those rates.

## 11. Reliability Strategy

The architecture uses several reliability mechanisms:

### Timeout

Prevents a slow dependency from holding resources indefinitely.

### Retry

Used selectively for transient failures.

### Circuit Breaker

Prevents repeated calls to an unhealthy external dependency.

### Idempotency

Prevents duplicate business operations during retries or repeated requests.

### Durable Events

Protects important asynchronous work from temporary downstream outages.

### Dead Letter Queue

Isolates messages that cannot be successfully processed.

### Compensation

Allows operations such as inventory reservation to be released when a subsequent operation, such as payment, fails.

### Reconciliation

Provides a mechanism to detect and repair inconsistencies after failures.

## 12. Deployment Approach

The proposed deployment uses horizontally scalable application instances behind a load balancer.

A conceptual deployment is:

```text
Users
  |
CDN / WAF
  |
Load Balancer
  |
  +-------------------+
  |                   |
API Instance       API Instance
  |                   |
  +---------+---------+
            |
     Application Layer
            |
    +-------+--------+
    |                |
 PostgreSQL        Redis
    |
Durable Event Infrastructure
    |
Workers / Recovery / Notifications
```

The exact infrastructure technology can vary between development, staging and production environments.

## 13. Observability

The system should expose operational visibility for:

- Request rate
- Request latency
- Error rate
- Admission/rejection rate
- Inventory reservation failures
- Payment failures
- Order creation failures
- Order conversion
- Queue/event backlog
- Recovery activity

The architecture should support:

- Structured logs
- Metrics
- Distributed tracing
- Alerts

## 14. Main Architectural Bottlenecks

The most important potential bottlenecks are:

### Inventory Database

Concurrent reservation requests can create contention around the limited inventory rows.

### Payment Gateway

An external payment provider can introduce latency, failures or timeouts.

### Order Service

A temporary outage must not cause successful payments to become permanently orphaned.

### Message Processing

A large flash-sale event volume can create backlog pressure.

### Database Capacity

Transactional throughput must remain sufficient for the inventory and order workloads.

## 15. Key Architectural Trade-offs

### SQL vs NoSQL

A relational transactional database is preferred for the core inventory, reservation, payment and order consistency requirements.

### Synchronous vs Asynchronous

Synchronous communication is appropriate when an immediate business result is required.

Asynchronous communication is preferred for durable downstream processing, notifications and recovery.

### Optimistic vs Pessimistic Concurrency

The inventory design can use atomic conditional updates and version/concurrency checks to protect stock without unnecessarily locking large portions of the system.

### Cache vs Database Consistency

Redis can reduce read pressure and support fast-path operations, but the transactional inventory source of truth remains PostgreSQL.

### Reliability vs Latency

Additional retries, durability and recovery mechanisms can increase latency. The architecture therefore uses bounded retries, admission control and asynchronous processing where appropriate.

## 16. Prototype Validation Reference

The current prototype has been used to validate critical architectural assumptions.

Observed simulation results include:

| Metric | Observed |
|---|---:|
| Ingress requests | 10,000 |
| Successful reservations | 100 |
| Oversold units | 0 |
| Duplicate orders | 0 |
| Payment successes | 94 |
| Payment failures | 6 |
| Recovered orders | 94 |
| Final available stock | 6 |
| Automated tests | 14 passed |

The observed average ingress latency in the current simulation was:

```text
62.6 ms
```

This is above the previously stated `<10 ms` latency target and therefore remains an optimization area.

These results represent prototype validation evidence and should not be interpreted as production capacity or production SLA measurements.

## 17. HLD Diagrams

The following diagrams provide the visual representation of this architecture.

### System Context

`01_System_Context.png`

Shows SALESTORM's relationship with customers and external systems.

### High-Level Architecture

`02_High_Level_Architecture.png`

Shows the major logical components and the end-to-end purchase path.

### Container Architecture

`03_Container_Architecture.png`

Shows the major applications/data stores and their communication boundaries.

### Deployment Architecture

`04_Deployment_Architecture.png`

Shows how the logical architecture can be deployed and horizontally scaled.

## 18. Architecture Principle

The primary design principle is:

> Protect the scarce inventory resource first, control traffic before it reaches the transactional boundary, and make payment/order processing recoverable through idempotency and durable asynchronous processing.
