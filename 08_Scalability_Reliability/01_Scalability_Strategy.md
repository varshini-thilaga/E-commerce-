# Scalability Strategy

This document outlines how the SALESTORM platform dynamically scales to handle flash sales.

## 1. Horizontal Application Scaling
All API services (Reservation, Payment, Order) are strictly **stateless application instances**. They store no local session data, allowing Kubernetes or Auto-Scaling Groups to dynamically add or remove instances based on CPU or request queue metrics.

## 2. Load Balancing
Traffic is distributed evenly across application nodes via a Layer 7 Application Load Balancer.

## 3. Admission Control
To prevent database contention, scaling compute alone is insufficient. The most critical scaling strategy is dropping excess load *before* it hits the database. Fast admission control (via Redis) sheds load instantly.

## 4. Key Strategy Flow
```text
Traffic spike (e.g., 500k req/s)
    ↓
CDN/WAF (Blocks malicious traffic/bots)
    ↓
Load Balancer
    ↓
API Gateway
    ↓
Admission Control (Load sheds excess traffic)
    ↓
Only accepted traffic reaches transactional services
```

## 5. Redis Caching
Redis is utilized for distributed admission control (e.g., Token Bucket counters) and caching hot product metadata, vastly reducing read load on the database.

## 6. Asynchronous Workers
Work that does not require an immediate synchronous response to the user (e.g., Order creation, Email notifications, Reservation expiry sweeps) is offloaded to asynchronous workers scaling independently based on queue depth.

## 7. Database Scaling
- **Read/write separation:** Write queries go to the Primary PostgreSQL instance, while catalog reads go to Read Replicas.
- **Connection Pooling:** PgBouncer or equivalent pools manage connections to prevent exhausting database memory.

## 8. Queue/Event Buffering
Message queues buffer traffic between domains (e.g., Payment -> Order). If the Order service slows down, messages simply queue up safely rather than dropping.

## 9. Bottleneck Analysis
- **Inventory database row contention:** The hottest bottleneck. 10,000 requests trying to update the exact same row (Product 100) will lock. We manage this via admission control and optimistic/conditional updates rather than throwing hardware at it.
- **Payment dependency:** External APIs have strict rate limits and network latency.
- **Order Service:** Subject to downstream bottlenecks.
- **Event processing backlog:** Handled by scaling consumers.
- **Database write capacity:** Handled via vertical scaling or eventual sharding.

## 10. Workload Context
- **Normal workload:** The architecture easily handles ~10k req/s via horizontal scaling.
- **Flash-sale design scenario:** We reason about spikes up to ~500k req/s primarily through aggressive Layer 4/7 load shedding and admission control. 
*(Note: We do NOT claim the prototype has demonstrated 500k req/s, but the architectural layers are designed to manage it).*
