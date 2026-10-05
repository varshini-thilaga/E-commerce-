# Adapter Pattern - External Payment Integration

## Problem
External payment providers (e.g., Stripe, PayPal, Adyen) expose highly specific, proprietary REST APIs with unique request structures and response schemas. If the SALESTORM core domain uses these external formats directly, our business logic becomes tightly coupled to a third-party vendor. Replacing or upgrading the vendor would require a massive rewrite of our core code.

## Structure (Conceptual)

```text
PaymentService
      |
      v
PaymentProvider (SALESTORM domain interface)
      |
      v
PaymentProviderAdapter
      |
      v
External Payment API (Stripe, etc.)
```

## Responsibilities

The Adapter translates between the internal SALESTORM payment contract and the provider-specific API contract.
- It maps the internal `PaymentRequest` to the external vendor's JSON payload.
- It parses the vendor's JSON response and translates it back into the internal `PaymentResponse`.

## Benefits
- Isolates the core domain from third-party volatility.
- Makes it trivial to swap vendors.

## Trade-offs
- Adds a translation layer that requires mapping code and maintenance.

## Failure and Timeout Considerations
A critical function of the Adapter is translating vendor-specific HTTP error codes and timeout exceptions into standard domain errors. Provider-specific errors (e.g., `stripe_invalid_api_key` or `connection_reset_by_peer`) should not leak into the core payment domain unnecessarily. The adapter must catch these and return a standardized `PaymentFailed` or `PaymentTimeout` response that the `PaymentService` understands.
