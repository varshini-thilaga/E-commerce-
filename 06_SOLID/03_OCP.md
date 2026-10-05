# Open/Closed Principle

Software components should be open for extension but closed for unnecessary modification. In SALESTORM, this ensures that as the business grows, we can add new capabilities without destabilizing the battle-tested core transaction engines.

## SALESTORM Payment Example

The most prominent application of OCP in our architecture is the payment provider integration.

```text
PaymentService (Core orchestration, closed for modification)
      |
      v
PaymentStrategy / PaymentProvider abstraction (The extension point)
      |
      +---- MockPaymentProvider (Used in current validation simulation)
      |
      +---- FutureExternalPaymentProvider (e.g., Stripe, Adyen)
```

### Explanation
The `PaymentService` contains the critical logic for idempotency checks, state transitions, and event emission. Adding another payment provider should not require rewriting this core checkout/payment orchestration. Instead, we implement a new class adhering to the `PaymentProvider` interface.

## Other Possible Extensions

- **Different admission-control strategies:** The prototype currently uses a Token Bucket algorithm via Redis. We could extend this to use a Leaky Bucket or predictive ML-based shedding by implementing a new strategy, without modifying the `FlashSaleAdmissionController` facade.
- **Different notification channels:** We could add SMS or Push Notification providers without touching the core `OrderService`.

## Design Trade-offs
Do not over-abstract. Too many abstractions can increase complexity, obfuscate code execution paths, and impact latency. Therefore, only meaningful variation points (like external API dependencies or highly volatile algorithms) receive interfaces/strategies in the SALESTORM design. Simple internal utilities do not.
