# Security Architecture

This document defines the strict trust boundaries and security controls protecting the SALESTORM platform.

## 1. Trust Boundaries

- **Internet Edge:** Untrusted. Protected by CDN/WAF.
- **Customer/Client Boundary:** Untrusted. External devices calling the API Gateway.
- **API Gateway Boundary:** Semi-trusted. Validates identity and drops malformed/malicious traffic.
- **Internal Service Boundary:** Trusted for execution, but strictly authenticated via Service-to-Service tokens.
- **Database Boundary:** Highly trusted. Only accessible by internal application services via secure networking.
- **External Payment Provider Boundary:** Untrusted external dependency. Accessed via secure, outbound-only HTTPS.
- **Administrative/Operator Boundary:** Highly restricted. Requires MFA and strict RBAC.

## 2. Security Controls

- **HTTPS/TLS:** Mandatory for all external and internal network boundaries.
- **Authentication:** Answers "Who are you?" via signed Bearer tokens (JWT).
- **Authorization:** Answers "What are you allowed to do?" enforced at the object level.
- **Input Validation:** Strict JSON schema validation at the Gateway.
- **Rate Limiting / WAF:** Blocks brute force and DDoS at the edge.
- **Secure Headers:** Enforces HSTS, CSP, and prevents framing.
- **Least Privilege:** Services run with minimal IAM permissions and restricted database roles.
- **Secrets Management:** Passwords and keys are never hardcoded.
- **Secure Error Handling:** Prevents stack traces from leaking to clients.

## 3. Roles and Object-Level Authorization

It is insufficient to merely authenticate a user. The application strictly enforces object-level authorization (e.g., verifying that the user requesting `GET /api/v1/orders/{orderId}` is the actual owner of that order).

### Examples:
**Customer:**
- Browse products (Public)
- Manage own cart (Authorized to `cartId`)
- Create own reservations
- View own orders (Authorized to `orderId`)

**Administrator/Operator:**
- Operational dashboards
- Inventory administration (e.g., seeding stock)
- Observability access

Do not allow users to access another customer's resources merely by changing an ID (preventing Insecure Direct Object Reference - IDOR).
