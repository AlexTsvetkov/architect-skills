# Lesson 08: Observability & Reliability Engineering

> **Prerequisites:** Lessons 01-07  
> **Time estimate:** 6-8 hours  
> **Parallel reading:** "Release It!" (2nd Ed) — Michael Nygard

---

## Table of Contents

1. [Three Pillars of Observability](#1-three-pillars)
2. [Structured Logging](#2-structured-logging)
3. [Distributed Tracing](#3-distributed-tracing)
4. [Metrics and Monitoring](#4-metrics-and-monitoring)
5. [SLIs, SLOs, SLAs](#5-slis-slos-slas)
6. [Chaos Engineering](#6-chaos-engineering)
7. [Incident Management](#7-incident-management)
8. [Exercises](#8-exercises)
9. [Self-Check Questions](#9-self-check-questions)
10. [Answers](#10-answers)

---

## 1. Three Pillars of Observability

```
                    OBSERVABILITY
           ┌────────────┼────────────┐
       ┌───┴───┐   ┌───┴───┐   ┌───┴───┐
       │ LOGS  │   │METRICS│   │TRACES │
       │What   │   │How is │   │Where  │
       │happened│   │the    │   │is the │
       │(events)│   │system │   │request│
       │       │   │doing? │   │going? │
       └───────┘   └───────┘   └───────┘
```

- **Monitoring** tells you WHEN something is wrong (alerts)
- **Observability** tells you WHY something is wrong (investigation)

---

## 2. Structured Logging

### JSON Logging

```
❌ Plain text:
2024-03-15 10:30:45 INFO OrderService - Order 123 created for customer 456

✅ Structured JSON:
{
  "timestamp": "2024-03-15T10:30:45.123Z",
  "level": "INFO",
  "service": "order-service",
  "traceId": "abc123def456",
  "spanId": "789012",
  "message": "Order created",
  "orderId": "123",
  "customerId": "456",
  "totalAmount": 99.99
}
```

### Log Aggregation Pipeline

```
App (stdout) → Fluentd/Filebeat → Elasticsearch → Kibana
                 (collect)          (store/index)   (visualize)

Alternative: App → CloudWatch / Azure Monitor / GCP Cloud Logging
```

### Best Practices

| Do | Don't |
|----|-------|
| Log to stdout/stderr | Log to files inside containers |
| Use structured JSON | Use plain text |
| Include traceId for correlation | Log without request context |
| Log business events | Log only technical events |
| Redact PII and secrets | Log credit cards, passwords, tokens |

---

## 3. Distributed Tracing

### The Problem

```
User: "My order took 15 seconds"
Which service was slow out of 5 in the chain?

With tracing:
Trace: abc-123
├── Span 1: API Gateway (50ms)
│   └── Span 2: Order Service (12,000ms) ← BOTTLENECK
│       ├── Span 3: Inventory Service (200ms)
│       ├── Span 4: Payment Service (11,500ms) ← ROOT CAUSE
│       │   └── Span 5: External Payment API (11,400ms)
│       └── Span 6: Email Service (100ms)
```

### OpenTelemetry with Spring Boot

```yaml
management:
  tracing:
    sampling:
      probability: 1.0  # 100% in dev, lower in prod
  otlp:
    tracing:
      endpoint: http://jaeger:4318/v1/traces
```

Trace ID propagated via `traceparent` HTTP header automatically.

### Tracing Tools

| Tool | Type |
|------|------|
| Jaeger | Open-source (CNCF graduated) |
| Zipkin | Open-source |
| Grafana Tempo | Storage backend for Grafana |
| AWS X-Ray / Azure Monitor | Managed cloud services |

---

## 4. Metrics and Monitoring

### Four Golden Signals (Google SRE)

| Signal | Description | Example Metric |
|--------|-------------|----------------|
| **Latency** | Time to serve request | `http_request_duration_seconds` |
| **Traffic** | Request volume | `http_requests_total` |
| **Errors** | Failure rate | `http_requests_total{status="5xx"}` |
| **Saturation** | Resource fullness | CPU %, memory %, queue depth |

### RED Method (for services)

**R**ate, **E**rrors, **D**uration — the three things to measure for every service.

### USE Method (for resources)

**U**tilization, **S**aturation, **E**rrors — for CPU, memory, disk, network.

### Prometheus + Grafana Stack

```java
// Custom business metrics with Micrometer
@Component
public class OrderMetrics {
    private final Counter ordersCreated;
    private final Timer orderProcessingTime;

    public OrderMetrics(MeterRegistry registry) {
        this.ordersCreated = Counter.builder("orders.created")
            .tag("channel", "web").register(registry);
        this.orderProcessingTime = Timer.builder("orders.processing.time")
            .register(registry);
    }
}
```

### Alerting Rules

```yaml
# Prometheus alert
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[5m]) 
        / rate(http_requests_total[5m]) > 0.05
  for: 5m
  labels: { severity: critical }
  annotations:
    summary: "Error rate above 5% for 5 minutes"

- alert: HighLatency
  expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
  for: 5m
  labels: { severity: warning }
```

---

## 5. SLIs, SLOs, SLAs

### Definitions

```
SLI (Service Level Indicator): What you measure
  → "99.2% of requests completed in < 200ms"

SLO (Service Level Objective): Internal target for SLI  
  → "99.5% of requests should complete in < 200ms"

SLA (Service Level Agreement): Contractual promise with penalties
  → "If availability < 99.9%, customer gets service credits"
```

### Error Budgets

```
SLO: 99.9% availability per month
Error budget: 0.1% = ~43 minutes downtime/month

Current availability: 99.95%
  → Budget used: 50% → safe to deploy new features

Current availability: 99.85%
  → Budget exhausted → freeze deployments, focus on reliability
```

Error budgets align engineering priorities: when budget is healthy, innovate; when it's low, stabilize.

---

## 6. Chaos Engineering

### Principles

1. Define "steady state" (normal system behavior)
2. Hypothesize that steady state continues during experiment
3. Introduce real-world failures (kill nodes, add latency, fill disk)
4. Observe: did the system degrade gracefully?
5. Fix what breaks

### Common Experiments

| Experiment | Tests | Tools |
|-----------|-------|-------|
| Kill random pod | Auto-recovery, health checks | Chaos Monkey, Litmus |
| Add network latency (500ms) | Timeouts, circuit breakers | tc (Linux), Gremlin |
| Fill disk to 95% | Log rotation, alerts | stress-ng |
| Simulate DNS failure | Service discovery fallback | iptables |
| Slow down database | Connection pool, query timeouts | Toxiproxy |

### Game Days

Scheduled chaos experiments with the full team. Run experiments in staging (or production with safeguards). The goal is to find weaknesses before they find you.

---

## 7. Incident Management

### Incident Response Process

```
1. DETECT → Monitoring alerts, customer reports
2. TRIAGE → Assess severity, assign incident commander
3. MITIGATE → Restore service (rollback, scale up, failover)
4. RESOLVE → Fix root cause
5. POST-MORTEM → Blameless analysis, action items
```

### Blameless Post-Mortems

Template:
```
Title: [Incident Title]
Date: [Date]
Duration: [X hours Y minutes]
Severity: [P1/P2/P3]
Impact: [What was affected, how many users]

Timeline:
  10:30 — Alert fired: error rate > 5%
  10:32 — On-call engineer acknowledged
  10:45 — Root cause identified: DB connection pool exhaustion
  10:50 — Mitigation: increased pool size, restarted pods
  11:00 — Service restored

Root Cause: Connection leak in new payment integration code.

Action Items:
  [ ] Add connection pool monitoring and alerts
  [ ] Add connection leak detection in CI tests
  [ ] Review all DB connection handling code
```

**Key rule:** Blameless. Focus on systems and processes, not people.

---

## 8. Exercises

### Exercise 1: Observability Strategy

Design an observability strategy for a microservices platform with 8 services:
1. Logging: format, aggregation, retention policy
2. Metrics: what to measure per service, alerting rules
3. Tracing: sampling strategy, what to trace
4. Dashboard: design a Grafana dashboard layout

### Exercise 2: SLO Definition

Define SLOs for an e-commerce order service:
1. Define 3 SLIs with specific measurement methods
2. Set SLO targets with justification
3. Calculate the error budget for each
4. Define alerting based on burn rate

### Exercise 3: Chaos Experiment

Design 5 chaos experiments for a payment processing service:
1. What failure you're introducing
2. What you expect to happen (hypothesis)
3. What you'll measure
4. What a "pass" vs "fail" looks like
5. Safeguards (how to stop if things go wrong)

---

## 9. Self-Check Questions

1. What are the three pillars of observability and how do they complement each other?
2. Why should cloud-native applications use structured JSON logging to stdout?
3. Explain distributed tracing. What is a trace and what is a span?
4. What are the Four Golden Signals and the RED method?
5. Explain SLI, SLO, and SLA. How do error budgets work?
6. What is chaos engineering? Name three experiments and what they test.
7. What makes a good post-mortem?

---

## 10. Answers

### Answer 1
**Logs:** Detailed record of discrete events — what happened and when. Best for debugging specific issues. **Metrics:** Aggregated numerical data over time — how the system is performing. Best for alerting and dashboards. **Traces:** Request flow across services — where time is spent. Best for diagnosing latency and finding bottlenecks. They complement: metrics alert you to a problem (high latency), traces help you find which service is slow, logs help you understand why that service is slow.

### Answer 2
**Stdout:** Containers are ephemeral; files are lost when pods restart. The platform (Kubernetes, CloudWatch) captures stdout and routes it. No disk management needed. **Structured JSON:** Enables searching/filtering by any field in log aggregation systems (Elasticsearch, CloudWatch). A plain text log `"Order 123 failed"` can't be filtered by orderId. A JSON log with `"orderId": "123"` can be queried instantly.

### Answer 3
A **trace** represents the entire lifecycle of a request across all services. A **span** represents a single unit of work within a trace (one service call, one DB query). Spans have: service name, operation, start time, duration, status, parent span ID. Spans are nested — a span in Service A creates child spans when it calls Services B and C. The trace ID is propagated via HTTP headers (W3C `traceparent`) so all services link their spans to the same trace.

### Answer 4
**Four Golden Signals** (Google SRE): Latency (request duration), Traffic (request rate), Errors (failure rate), Saturation (resource utilization). **RED Method** (for services): Rate (requests/sec), Errors (failed requests/sec), Duration (latency distribution — p50, p95, p99). RED is a subset of Golden Signals focused on what matters most for services. USE method (Utilization, Saturation, Errors) is the complement for infrastructure resources.

### Answer 5
**SLI:** A measured metric of service quality (e.g., "99.2% of requests < 200ms"). **SLO:** An internal target for the SLI (e.g., "99.5% of requests should be < 200ms"). SLO is stricter than SLA to provide a buffer. **SLA:** A contractual promise with business consequences (e.g., "99.9% uptime or service credits"). **Error budget:** The allowed amount of unreliability. If SLO = 99.9%, error budget = 0.1% (~43 min/month). When budget is healthy, teams can deploy and experiment. When exhausted, teams focus on reliability. This creates a quantitative framework for balancing innovation and stability.

### Answer 6
Chaos engineering is the practice of deliberately introducing failures into a system to discover weaknesses before they cause incidents. (1) **Kill a pod** — tests auto-recovery, health checks, and zero-downtime design. (2) **Add network latency** — tests timeouts, circuit breakers, and retry policies. (3) **Simulate database failure** — tests fallback behavior, circuit breakers on DB connections, and cache resilience.

### Answer 7
A good post-mortem is: **Blameless** — focuses on systems and processes, not individuals. **Timely** — written within 48 hours while memory is fresh. **Thorough** — includes timeline, root cause, impact assessment. **Actionable** — concrete action items with owners and deadlines. **Shared** — published to the organization so others learn. **Followed up** — action items are tracked to completion, not forgotten.

---

*Previous: [07 - API Design](07-api-design.md)*  
*Next: [09 - Security](09-security.md)*