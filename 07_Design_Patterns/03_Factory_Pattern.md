# Factory Pattern - Payment Strategy Creation

## Problem
While the Strategy Pattern abstracts the execution of payment processing, the system still needs a way to instantiate the correct strategy implementation based on runtime parameters (such as configuration, payment method type, or environment). Scattering this instantiation logic (`new MockPaymentProvider()` vs `new ExternalPaymentProvider()`) throughout the business logic violates the Single Responsibility Principle and couples the application to concrete classes.

## Structure
```text
Client / Dependency Injector
      |
      v
PaymentFactory (Proposed architectural component)
      |
      +---- creates ---> MockPaymentProvider
      |
      +---- creates ---> ExternalPaymentProvider
```

## SALESTORM Application
The `PaymentFactory` centralized creation component selects and instantiates the correct `PaymentStrategy` implementation based on:
- System environment variables (e.g., `ENV=test` yields `MockPaymentProvider`).
- The specific payment method requested by the checkout payload.
- Active provider configuration (e.g., routing traffic to an alternative provider if one is degraded).

## Benefits
- Centralizes creation logic, preventing it from leaking into the `PaymentService` or `CheckoutFacade`.
- Simplifies dependency injection.
- Makes it trivial to add a new provider to the system by simply registering it with the factory.

## Trade-offs
- Adds an additional layer of abstraction (the factory class itself).

## Relationship to Strategy
The Factory is responsible for *creation* (which strategy to use), while the Strategy pattern is responsible for *execution* (how the strategy works). 

**Important Note:** If the prototype and final production environment only strictly necessitate a single payment provider, a simple parameterized factory (or basic Dependency Injection container wiring) is entirely sufficient. A complex "Abstract Factory" pattern is unnecessary and should be avoided.
