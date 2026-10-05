# Design Pattern Mapping in SALESTORM

This document maps software design patterns to their specific applications within the SALESTORM platform.

*Note: Classes and interfaces listed below include both those implemented in the current prototype and those proposed as abstractions in the Low-Level Design (LLD).*

---

### 1. Strategy Pattern
- **Problem:** Processing payments requires distinct logic depending on the payment method or external provider (e.g., Mock Provider vs. Stripe).
- **Why it fits:** It encapsulates the specific payment logic into swappable classes without altering the `PaymentService` flow.
- **Relevant classes:** `PaymentStrategy` (LLD abstraction), `MockPaymentProvider` (Prototype), `ExternalPaymentProvider` (Proposed).

### 2. Factory Pattern
- **Problem:** Instantiating the correct `PaymentStrategy` based on user input or configuration clutters the core orchestration logic.
- **Why it fits:** Centralizes the creation and selection logic.
- **Relevant classes:** `PaymentFactory` (Proposed architectural component).

### 3. State Pattern
- **Problem:** Orders and Reservations transition through rigid lifecycles (e.g., `CREATED` -> `RESERVED` -> `EXPIRED`). Invalid transitions must be prevented.
- **Why it fits:** Ensures state-specific rules are enforced and guards against illegal transitions.
- **Relevant classes:** `OrderStateMachine`, `ReservationState` (LLD design abstractions).

### 4. Observer Pattern
- **Problem:** The Order Service needs to act when a payment succeeds, but coupling them synchronously risks cascading failures.
- **Why it fits:** Allows asynchronous notification. The Payment Service publishes an event; interested consumers react independently.
- **Relevant classes:** `EventPublisher`, `EventConsumer` (Prototype implementation).

### 5. Adapter Pattern
- **Problem:** External payment APIs have proprietary request/response formats that do not match the internal SALESTORM domain models.
- **Why it fits:** Translates the external API contract into the standard `PaymentStrategy` contract.
- **Relevant classes:** `PaymentProviderAdapter` (Proposed architectural component).

### 6. Facade Pattern
- **Problem:** A client wants to "Checkout", which requires orchestrating inventory reservations, payment processing, and order creation.
- **Why it fits:** Hides the complexity of multiple subsystem interactions behind a single, unified entry point.
- **Relevant classes:** `CheckoutFacade` (Proposed architectural component).

### 7. Repository Pattern
- **Problem:** Business services need to save data without hardcoding SQL queries or being tightly coupled to PostgreSQL specifics.
- **Why it fits:** Provides a collection-like interface to access domain objects while hiding database mechanics.
- **Relevant classes:** `InventoryRepository`, `PaymentRepository`, `OrderRepository` (LLD design abstractions).

### 8. Circuit Breaker Pattern
- **Problem:** If a payment provider goes offline, continually attempting synchronous network calls will exhaust threads and memory, crashing the `PaymentService`.
- **Why it fits:** Fails fast when the dependency is unhealthy, preventing cascading failure.
- **Relevant classes:** `CircuitBreaker` (Proposed architectural component).
