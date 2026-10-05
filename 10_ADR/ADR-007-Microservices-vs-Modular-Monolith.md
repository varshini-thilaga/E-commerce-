# ADR-007: Microservices vs. Modular Monolith

## Status

Accepted

## Context

SALESTORM is designed as a highly scalable architecture. Typically, this implies a distributed Microservices architecture. However, deploying a complex distributed system (with Kubernetes, Istio, Kafka) during a hackathon validation phase introduces massive operational overhead.

## Decision

The SALESTORM architecture should preserve clear logical service boundaries (modular monolith design), while the prototype may implement those boundaries in a simpler deployment model.

**Logical architecture vs. Prototype deployment:**
The system is conceptually designed as decoupled services (Inventory, Payment, Order), but the local prototype compiles and runs these modules within a unified application boundary (a Modular Monolith) to facilitate testing and validation.

## Alternatives Considered

**Full Microservices Prototype:**
- *Pros:* Demonstrates independent scaling perfectly.
- *Cons:* Extremely high deployment complexity. Network failures between local Docker containers obfuscate the actual business logic validation. Development speed is severely degraded.

## Trade-offs

**What do we gain?**
- Fast development speed and simplified observability for the hackathon prototype.
- By maintaining strict interface boundaries (SOLID principles) between the modules, the modular monolith can be easily broken apart into true microservices when production scaling demands it.

**What do we give up?**
- We cannot independently scale the Payment module separately from the Inventory module in the current deployment. A memory leak in one module will crash the entire prototype application.

## Consequences

**Positive:**
- Focus remains on proving the core invariants (inventory correctness, concurrency handling, recovery) rather than fighting infrastructure orchestration.

**Negative:**
- Evaluators must understand the difference between the architectural design (which is distributed) and the prototype deployment (which is unified). We do not force unnecessary distributed infrastructure into the prototype merely to make the diagram look like a production platform.

## Revisit Conditions

Revisit this decision when the platform moves to a staging environment where actual independent scaling of the API edges is required to handle sustained multi-tenant traffic.
