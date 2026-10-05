# Audit and Security Logging

Audit logging is explicitly separated from normal application logging to ensure security and compliance tracking.

## Purpose

- **Application logs answer:** "What did the system do internally?" (e.g., executing a database query, encountering a timeout).
- **Security logs answer:** "Did anything suspicious or unauthorized happen?" (e.g., repeated failed logins, WAF blocks).
- **Audit logs answer:** "Who did what, when, to which resource, and what happened?" (e.g., Admin changed inventory stock from 100 to 0).

## Audit Events
Audit events capture critical business and security transitions:
- Login success/failure
- Authorization failure
- Reservation creation
- Reservation release
- Payment initiation
- Payment success/failure
- Order creation
- Order cancellation
- Administrative inventory changes
- Recovery/reconciliation actions

## Useful Fields
Structured audit logs must contain:
- `timestamp`: ISO-8601 UTC time.
- `eventId`: Unique identifier for the audit record.
- `actorId`: User ID, Admin ID, or System Principal.
- `actorType`: Customer, Admin, System.
- `action`: E.g., `CREATE_RESERVATION`.
- `resourceType`: E.g., `Inventory`.
- `resourceId`: E.g., `PROD-100`.
- `result`: `SUCCESS` or `FAILURE`.
- `correlationId`: To trace the action across systems.
- `requestId`: The specific HTTP request ID.
- `source/service`: E.g., `ReservationService`.
- `reason/errorCode`: If the action failed.

## Crucial Restrictions
**Never log the following under any circumstances:**
- Passwords or hashes
- Authentication tokens (JWTs, session IDs)
- Raw payment credentials (credit card numbers, CVVs)
- Secrets, private keys, or API keys
- Unnecessary sensitive personal information (PII)

## Retention and Immutability
Audit logs should be shipped to a separate, append-only storage system (e.g., AWS CloudTrail or a dedicated SIEM) to provide tamper resistance. Retention policies are dictated by legal and compliance requirements (e.g., keeping financial transaction audit logs for 7 years).
