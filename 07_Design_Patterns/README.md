# SALESTORM Design Patterns

## 1. Purpose
This folder documents the software design patterns utilized in the STORMSHIELD Flash-Sale Resilience Platform architecture. Rather than providing generic tutorials, this documentation strictly ties established software design patterns to concrete SALESTORM components and business problems.

## 2. Pattern Selection Philosophy
Patterns are not applied merely to inflate complexity. They are selected based on actual points of variation, lifecycle complexity, critical integration boundaries, and strict reliability requirements. Where a simple implementation suffices, we avoid over-engineering.

## 3. Pattern Summary

| Pattern | SALESTORM Application | Main Benefit |
| --- | --- | --- |
| **Strategy** | Payment strategy/provider | Replace algorithms/providers |
| **Factory** | Payment strategy creation | Centralized creation/selection |
| **State** | Order/reservation lifecycle | Controlled state transitions |
| **Observer** | Domain events | Loose coupling between consumers |
| **Adapter** | External payment API | Isolate provider-specific interfaces |
| **Facade** | Checkout orchestration | Simplify multi-service workflow |
| **Repository** | Persistence | Isolate database access |
| **Circuit Breaker** | External/dependent calls | Prevent cascading failures |

## 4. Relationship to SOLID
These patterns are the practical execution of SOLID principles defined in Folder `06_SOLID`. For example, the Strategy pattern directly satisfies the Open/Closed Principle (OCP), while the Repository pattern enforces the Dependency Inversion Principle (DIP).

## 5. Relationship to LLD
The patterns documented here map directly to the classes, abstractions, and relationships established in the Low-Level Design (LLD). 

## 6. Reliability and Scalability Impact
Patterns like Circuit Breaker and Observer are critical for system reliability. The Observer pattern (via durable event brokers) allows the system to scale asynchronously, decoupling fast admission processes from slower downstream fulfillment operations.

## 7. Design Trade-offs
Every pattern introduces a degree of indirection and cognitive overhead. We explicitly document when a pattern is necessary (e.g., hiding external payment APIs behind an Adapter) versus when it might be overkill (e.g., using a complex State pattern hierarchy when a simple enum suffices for minor state tracking).
