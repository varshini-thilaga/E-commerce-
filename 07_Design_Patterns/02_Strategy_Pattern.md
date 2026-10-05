# Strategy Pattern - Payment Processing

## Problem
In SALESTORM, payment processing must support different provider implementations (e.g., Stripe, Adyen, or a simulated Mock Provider for hackathon validation) or distinct payment behaviors. Hardcoding multiple `if/else` statements for each provider inside the core payment orchestration logic creates brittle, tightly coupled code that is difficult to test and maintain.

## Structure
```text
PaymentService
      |
      v
PaymentStrategy (Abstraction)
      |
      +---- MockPaymentProvider (Prototype)
      |
      +---- ExternalPaymentProvider (Proposed architectural component)
```

## SALESTORM Classes
- `PaymentService` (Context)
- `PaymentStrategy` (Strategy Interface)
- `MockPaymentProvider` (Concrete Strategy)

## Flow
1. The `PaymentService` is invoked to process a payment.
2. It relies entirely on the `PaymentStrategy` interface abstraction to execute the payment.
3. The injected implementation (e.g., `MockPaymentProvider`) performs the actual work.
4. The strategy returns standard success, failure, or timeout outcomes according to the defined contract.

## Benefits
- Different implementations can be substituted dynamically without rewriting core payment orchestration.
- **Testability:** The core business rules in `PaymentService` can be tested purely by injecting a mock strategy.
- Directly satisfies the **Open/Closed Principle (OCP)** (open for new providers, closed to core modification) and **Dependency Inversion Principle (DIP)**.

## Trade-offs
- Increases the number of interfaces and classes.
- The interface contract must be broad enough to encompass the requirements of all possible strategies without leaking implementation details.

## When NOT to use it
If the platform will strictly only ever support a single internal ledger system without external integrations, extracting a strategy interface constitutes premature over-engineering.
