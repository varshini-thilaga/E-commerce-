# SALESTORM Security & Observability

## 1. Purpose
Security and Observability are cross-cutting concerns that span the entire SALESTORM architecture. This folder documents the security boundaries, threat mitigations, and operational visibility required to defend the system and monitor its health during extreme traffic spikes.

*(Note: API contracts are defined in Folder 05, and scalability mechanics in Folder 08. This folder focuses strictly on security controls and observability.)*

## High-Level Flow

```text
Customer
   |
   v
CDN / WAF
   |
   v
Load Balancer
   |
   v
API Gateway
   |
   +-------------------+
   |                   |
Authentication      Rate Limiting
Authorization       Validation
   |                   |
   +---------+---------+
             |
             v
        Application Services
             |
      +------+------+
      |             |
 PostgreSQL        Redis
      |
 Payment Provider
```
*Observability spans this entire path, tracking requests from the edge to the database.*

## Key Topics

- **Security Objectives & Trust Boundaries:** Defining where zero-trust principles apply.
- **Authentication and Authorization:** Validating identity and enforcing access controls.
- **API & Data Protection:** Securing payloads, secrets, and raw payment data.
- **Audit Logging:** Tracking "who did what, when, and to which resource."
- **Observability (Metrics, Logs, Distributed Tracing):** Tracking system health and business operations.
- **Alerting & Operational Dashboard:** Detecting and responding to incidents.

## Key Security Trade-offs
To maintain high throughput, deep content inspection (like anti-malware scanning) is offloaded to the CDN/WAF edge rather than performed synchronously in the core transactional flow. We accept the complexity of distributed tracing in order to gain visibility across asynchronous decoupling boundaries.
