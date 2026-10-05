# Observability Architecture

This document defines the high-level observability architecture for SALESTORM.

## Flow

```text
Services
   |
   +--> Metrics (Counters, Gauges, Histograms)
   |
   +--> Structured Logs (JSON format)
   |
   +--> Distributed Traces (Spans, Correlation IDs)
   |
   v
Observability Platform (e.g., Prometheus, Grafana, ELK, Datadog)
   |
   +--> Dashboards (Visualizing health)
   |
   +--> Alerts (Automated paging on anomalies)
   |
   +--> Incident Investigation (Querying logs/traces)
```

## Core Pillars

1. **Metrics:** Quantitative data used for real-time monitoring and alerting (e.g., "We are processing 10,000 req/s").
2. **Logs:** Immutable records of discrete events (e.g., "Request ID 123 failed due to timeout").
3. **Distributed Tracing:** Maps the flow of a single request across multiple microservices.

## Technical vs. Business Observability

It is critical to separate system health from business health.

**Technical Observability Examples:**
- CPU and memory utilization
- Endpoint latency (p99)
- HTTP error rates (5xx)
- Database query latency
- Redis cache hit/miss ratio
- Event queue backlog / Consumer lag

**Business Observability Examples:**
- Total reservations attempted vs. successful
- Payment success rate (conversion)
- Total orders created
- Reservation expiry count (abandoned carts)
- Order recovery count (how many orders required async fallback)

By monitoring both, operators can determine if a technical blip (e.g., Database latency spiking) is actually impacting the business (e.g., Order creation rate dropping).
