# Data Protection, Secrets, and Payment Security

## 1. Data Classification

Data within SALESTORM is classified to apply appropriate protections:
- **Customer data (PII):** Email, addresses (Encrypted at rest, strict access control).
- **Authentication tokens:** Handled transparently, verified cryptographically, short TTL.
- **Order information:** Business confidential.
- **Logs & Metrics:** Sanitized, no PII.

## 2. Secrets Management

Secrets (e.g., database passwords, external API keys, JWT signing keys) are:
- Injected via environment variables at runtime or retrieved from a Secure Secrets Manager.
- Periodically rotated.
- **Never included in documentation, source code, or logs.** 
*(Note: Any examples in documentation use safe placeholders like `<YOUR_API_KEY>` or `[REDACTED]`)*.

## 3. Encryption
- **In Transit:** All external and internal HTTP traffic is encrypted using TLS 1.2+.
- **At Rest:** Database volumes storing PII and order data utilize disk-level encryption (e.g., AES-256).

## 4. Payment Security (CRITICAL)

The system **does not unnecessarily store raw card or payment credentials**. SALESTORM adheres to PCI-DSS principles by offloading sensitive payment data handling to the external provider.

### Conceptual Flow:
```text
Checkout (Frontend securely tokenizes card with Stripe/Adyen directly)
   |
   v
Payment Service (Receives secure token, NOT the card number)
   |
   v
Payment Provider Adapter
   |
   v
External Payment Provider
```

*The local prototype uses a simulated payment provider, but the architecture dictates that production implementations utilize provider-specific tokenization.*

## 5. Data Minimization
The system only collects data strictly necessary for the business transaction. Payment references are retained for reconciliation, but unnecessary data is discarded.
