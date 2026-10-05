# Threats and Mitigations

The following threat model identifies SALESTORM-specific risks and architectural mitigations.

| Threat | Impact | Mitigation |
|---|---|---|
| **Credential theft** | Account takeover | Secure authentication/token handling via Identity Provider. |
| **Broken object authorization (IDOR)** | Access to another user's order | Strict object-level authorization in business logic. |
| **Duplicate purchase automation** | Inventory abuse / Bot hoarding | Idempotency constraints + admission control. |
| **Request flooding (DDoS)** | Resource exhaustion | WAF + rate limiting + Token Bucket admission control. |
| **SQL injection** | Data compromise | Strict parameterized queries / ORM usage. |
| **Sensitive data leakage** | Privacy/security breach | Data minimization + log redaction. |
| **Payment manipulation** | Financial impact / Free goods | Provider adapter + strict server-side validation of payment outcomes. |
| **Secret leakage** | System compromise | Secret manager + environment variables (no hardcoded keys). |
| **Replay of requests** | Duplicate business operation | Unique `Idempotency-Key` headers validated against database. |
| **Malicious event/message** | Incorrect state transitions | Schema validation + authenticated producers/consumers on message bus. |
| **Log injection** | Monitoring compromise / Fake trails | Structured/encoded JSON logging. |
| **Excessive API access** | Service degradation | Rate limits + authenticated authorization. |
| **Misconfiguration** | Public exposure of internal APIs | Secure defaults + infrastructure-as-code review. |

## SALESTORM Specific Considerations

- **Unrestricted Resource Consumption:** A flash sale is uniquely vulnerable to this. Mitigated by shedding load via HTTP 429 when the admission controller detects saturation.
- **Sensitive Business Flow Abuse:** Malicious actors may try to call the `/confirm` reservation endpoint directly without paying. Mitigated by restricting those endpoints to internal service-to-service communication only.
- **Unsafe External API Consumption:** If the payment provider is compromised, it could return malicious payloads. Mitigated by strict parsing and validation in the `PaymentProviderAdapter`.
