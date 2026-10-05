# SALESTORM - Traffic & Workload Assumptions

## 1. Normal Traffic
The system operates under a baseline "normal" traffic load of approximately 10,000 requests/sec across all endpoints.

## 2. Flash-Sale Traffic
The architectural design goals and reasoning are built to accommodate theoretical flash-sale spikes of up to approximately 500,000 requests/sec. 

## 3. Concurrent Purchase Burst
The specific challenge simulation workload targets 10,000 concurrent purchase requests arriving in a rapid burst targeting a single highly-contended SKU.

## 4. Inventory Model
The highly-contended flash-sale product has a strictly limited available stock of 100 units.

## 5. Payment Model
The simulated payment provider models real-world conversion rates:
- **Payment success:** 95% of initiated payments complete successfully.
- **Payment failure:** 5% of initiated payments fail (e.g., declined by bank, insufficient funds) or time out, triggering reservation release logic.

## 6. Duplicate Request Model
Due to aggressive client-side retries and network instability during a flash sale, it is assumed that 2% of incoming requests are duplicates (sharing the same Idempotency-Key).

## 7. Order Service Outage Model
To validate asynchronous resilience, the workload simulation injects a deliberate Order Service outage lasting 30 seconds during peak payment processing.

## 8. Failure Scenarios
The validation workload and demonstrations must explicitly simulate and survive the following scenarios:
- Successful purchase (happy path).
- Failed payment and subsequent reservation release.
- Duplicate Buy request handling.
- Payment succeeds but Order Service fails (event buffering).
- Zero inventory (graceful rejection of excess load).
- 50x traffic increase (simulated via 10,000 request burst).
- Database failure/rollback scenarios.
- Payment provider failure/timeout.

## 9. Scalability Assumptions
The design assumes stateless application nodes capable of horizontal scaling behind a load balancer, utilizing a centralized relational database as the primary bottleneck and consistency boundary.

## 10. Validation Workload
The current prototype is validated against the "challenge workload" (10,000 concurrent requests). 

*Important Distinction: The system successfully executing the challenge workload is proof of the architectural concepts and strict guarantees. It is not a production capacity claim that the current un-optimized local prototype has been benchmarked at 500,000 requests/sec.*
