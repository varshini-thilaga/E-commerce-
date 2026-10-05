# Liskov Substitution Principle

An implementation of an abstraction must remain safely substitutable wherever that abstraction is expected. Without this, the Open/Closed Principle breaks down because the client code would need `if/else` checks to handle specific implementation quirks.

## Application in SALESTORM

We look at the SALESTORM payment-provider abstraction as our primary example.

```text
PaymentProvider (LLD design abstraction)
    |
    +---- MockPaymentProvider (Currently implemented)
    +---- ExternalPaymentProvider (Proposed architectural component)
```

Both implementations must respect the exact same contract.

### Expected Behavior

Conceptually, the interface defines a behavior like:
```text
processPayment(request)
    -> success
    -> failure
    -> timeout/error according to the contract
```

Replacing the `MockPaymentProvider` with an `ExternalPaymentProvider` should not require the `PaymentService` to understand provider-specific behavior, such as specific HTTP status codes or custom exception types. The implementation must map those to standard domain responses.

### What Would Violate LSP?

If a future `ExternalPaymentProvider` implementation silently changes the meaning of success/failure (e.g., returning a "success" response when the payment is actually pending asynchronous clearance) or requires unsupported preconditions (e.g., demanding a field not present in the standard request object), it would break the abstraction contract. The orchestrating service would fail unpredictably.

### Trade-offs
We do not force inheritance where composition or simple interfaces are more appropriate. Overusing deep inheritance hierarchies often leads to LSP violations; therefore, SALESTORM relies on shallow interfaces (strategy pattern) for these substitutions.
