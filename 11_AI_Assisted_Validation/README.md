# SALESTORM AI-Assisted Validation

## 1. Purpose
This folder documents the empirical evidence gathered from the SALESTORM local prototype. **The prototype is supporting evidence for architectural decisions. It is NOT the primary deliverable.** The primary deliverable remains the system design.

## 2. What Was Validated
We validated the core correctness and resilience mechanisms of the architecture: inventory concurrency, load shedding (admission control), idempotency, and asynchronous failure recovery.

## 3. Validation Environment
The validation was performed against a modular-monolith prototype running locally, utilizing PostgreSQL for authoritative transactional state and Redis for fast-path admission.

## 4. Concurrency Scenario
We simulated 10,000 concurrent purchase requests competing for a single SKU with 100 available units.

## 5. Inventory Correctness
The prototype successfully protected the core business invariant (`available_quantity >= 0`), resulting in exactly 0 oversold units.

## 6. Payment Failure Handling
Simulated payment failures correctly triggered compensating transactions, releasing reserved inventory back to the available pool.

## 7. Order Service Recovery
A simulated 30-second Order Service outage validated the durable event buffer, successfully replaying and confirming orders upon recovery.

## 8. Automated Tests
The repository contains 14 automated test scenarios targeting specific architectural constraints, all of which pass.

## 9. Observed Results
- **10,000** concurrent requests.
- **100** successful unique reservations.
- **0** oversold units.
- **0** duplicate orders.
- **94** payment successes, **6** payment failures automatically released.
- **94** events buffered during Order Service outage.
- **94** orders replayed/confirmed after recovery.
- **6** units remaining available.
- **62.6 ms** average ingress latency.
- **14/14** automated tests passed.

## 10. Limitations
These are prototype validation results. They are NOT production benchmarks. We do not claim that 500k req/s was achieved, nor that <10 ms latency was achieved (as observed latency was 62.6 ms). We do not claim production-scale reliability or exactly-once delivery was proven. The validation provides supporting evidence for selected architectural assumptions.

## 11. AI Assistance
AI was utilized as an engineering accelerator during this project. Details are documented in `05_AI_Usage_Note_and_Prompt_Summary.md`.

## 12. Evidence Files
- `01_Prototype_Validation.md`
- `02_Load_Testing_and_Simulation.md`
- `03_Test_Results.md`
- `04_Failure_Scenario_Validation.md`
- `05_AI_Usage_Note_and_Prompt_Summary.md`
