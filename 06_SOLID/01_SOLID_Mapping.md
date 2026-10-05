# SOLID Mapping in SALESTORM

This document provides a detailed mapping of SOLID principles to actual and proposed SALESTORM architectural components.

## 1. Single Responsibility Principle (SRP)
- **Definition:** A class should have one, and only one, reason to change.
- **SALESTORM Components:** `InventoryService`, `PaymentService`, `IdempotencyService`
- **Application:** `InventoryService` is strictly responsible for managing atomic stock decrements. It does not handle HTTP routing, caching, or payment processing.
- **Why it matters:** Protects the critical consistency boundary.
- **Impact:** High testability (isolated atomic tests).
- **Trade-off:** Requires more classes and orchestration logic.

## 2. Open/Closed Principle (OCP)
- **Definition:** Entities should be open for extension, but closed for modification.
- **SALESTORM Components:** `PaymentProviderAdapter` (LLD design abstraction), `PaymentStrategy`
- **Application:** The checkout orchestration relies on a payment abstraction. We can add a `StripePaymentProvider` alongside the `MockPaymentProvider` (currently implemented in the prototype) without modifying the `PaymentService` class.
- **Why it matters:** External integrations change frequently; core business flow does not.
- **Impact:** High extensibility.
- **Trade-off:** Introduces indirection.

## 3. Liskov Substitution Principle (LSP)
- **Definition:** Subtypes must be substitutable for their base types.
- **SALESTORM Components:** `MockPaymentProvider`, `ExternalPaymentProvider` (Proposed)
- **Application:** Any class implementing the payment interface must adhere to the contract: returning standard success/failure/timeout responses rather than throwing unexpected fatal exceptions or altering global state.
- **Why it matters:** Ensures predictability during failure scenarios.
- **Impact:** High reliability and safe substitution.
- **Trade-off:** Requires strict interface enforcement and contract testing.

## 4. Interface Segregation Principle (ISP)
- **Definition:** Clients should not be forced to depend on methods they do not use.
- **SALESTORM Components:** `InventoryRepository`, `OrderRepository` (LLD abstractions)
- **Application:** Instead of a massive `CommerceRepository`, data access is segregated. The `InventoryService` only depends on the `InventoryRepository` interface to execute `reserve_stock()`, keeping it blind to `save_order()` methods.
- **Why it matters:** Prevents tight coupling across distinct database domains.
- **Impact:** Smaller contracts, better service boundaries.
- **Trade-off:** More boilerplate for dependency injection.

## 5. Dependency Inversion Principle (DIP)
- **Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions.
- **SALESTORM Components:** `ReservationService`, `PostgreSQL` implementation
- **Application:** The `ReservationService` (high-level policy) depends on an inventory repository abstraction. The PostgreSQL implementation (low-level detail) implements that abstraction. *Note: PostgreSQL remains the authoritative transactional source of truth, not replaced by Redis.*
- **Why it matters:** Allows testing without a live database and isolates the business logic from SQL dialects.
- **Impact:** Massive boost to testability.
- **Trade-off:** Adds abstraction layers that can obscure raw database performance tuning.
