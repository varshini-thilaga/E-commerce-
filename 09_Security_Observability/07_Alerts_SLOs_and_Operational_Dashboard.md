# Alerts, SLOs, and Operational Dashboard

## 1. Operational Dashboard Design

The operator dashboard provides real-time visibility into the flash sale.

**Sections:**
1. **Traffic:** Live ingress req/s, WAF blocks.
2. **Admission Control:** Accepted vs. Shed (429) requests.
3. **Inventory:** Live units available, units reserved.
4. **Payment:** Success/Failure rates.
5. **Orders:** Total confirmed orders.
6. **Messaging:** Queue lag and consumer health.
7. **Database:** Connection pool saturation, active queries.
8. **Redis:** Memory usage, hit rates.
9. **Security:** Auth failures, rate limit trips.
10. **Recovery:** Events in DLQ, successful replays.

## 2. Alerts

Alerts page the on-call engineers when the system deviates from expected behavior.

### HIGH PRIORITY (P1 - Page immediately)
- **Inventory invariant violation:** Unexpected negative stock detected.
- **Payment failure spike:** Provider integration broken.
- **Order recovery backlog:** Asynchronous processing has halted.
- **DLQ growth:** Poison messages are piling up.
- **Database unavailable / Redis unavailable:** Core infrastructure down.
- **Authentication/Authorization failure spike:** Potential credential stuffing or attack.
- **Suspicious traffic spike:** Volumetric DDoS attack.

### MEDIUM PRIORITY (P2 - Investigate soon)
- **Elevated latency:** Nearing SLO limits.
- **Increased reservation failures:** Likely normal when stock empties, but requires verification.
- **Increased payment timeout:** Provider is slow.
- **Increased event consumer lag:** Workers need scaling.
- **High DB connection usage:** Nearing pool limits.

## 3. SLOs (Service Level Objectives) vs. Business Invariants

It is critical to distinguish between goals and guarantees.

- **Business Invariant:** "No overselling." This is a mathematical correctness guarantee. It is NOT merely an SLO. If this fails, the system is fundamentally broken.
- **SLO (Service Level Objective):** "99.9% of requests succeed." This is a target reliability measurement over time.

### SALESTORM Targets
- **Availability target:** 99.9% uptime during the sale.
- **Error-rate target:** < 1% 5xx errors.
- **Payment success target:** > 90% (accounts for expected user-side funding issues).
- **Latency target:** < 10 ms ingress admission.

**Important Note on Prototype Validation:** 
The current prototype simulation observed approximately **62.6 ms** average ingress latency. Therefore, the <10 ms metric is explicitly a *design target*, not a demonstrated production result. 

*(Other Prototype evidence: 10,000 concurrent requests processed, 100 successful unique reservations, 0 oversold units, 94 payment successes, 94 orders successfully recovered from outages. These validate the architecture, but are not production security or performance certifications).*
