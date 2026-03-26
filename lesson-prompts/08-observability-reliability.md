# Prompt: Lesson 08 — Observability & Reliability Engineering

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 08: Observability & Reliability Engineering**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot with Micrometer, OpenTelemetry. Reference: Microsoft Monitoring Best Practices.

### Topic Coverage

1. **Three Pillars of Observability**
   - Monitoring vs Observability distinction
   - How logs, metrics, and traces complement each other

2. **Structured Logging**
   - Plain text vs structured JSON comparison
   - Log aggregation pipeline: App → Fluentd → Elasticsearch → Kibana
   - Best practices table (Do/Don't): stdout, JSON, traceId, PII redaction

3. **Distributed Tracing**
   - Problem illustration: 15-second request across 5 services (trace tree showing bottleneck)
   - OpenTelemetry with Spring Boot (YAML configuration)
   - W3C traceparent header propagation
   - Tracing tools table (Jaeger, Zipkin, Grafana Tempo, AWS X-Ray)

4. **Metrics and Monitoring**
   - Four Golden Signals (Google SRE): latency, traffic, errors, saturation
   - RED Method for services
   - USE Method for resources
   - Prometheus + Grafana: Custom Micrometer metrics code (Counter, Timer)
   - Prometheus alerting rules YAML (HighErrorRate, HighLatency)

5. **SLIs, SLOs, SLAs**
   - Clear definitions with examples
   - Error budgets: calculation, decision framework (innovate vs stabilize)
   - Error budget policy alignment

6. **Chaos Engineering**
   - 5 principles
   - Common experiments table (5 experiments with tools)
   - Game Days concept

7. **Incident Management**
   - 5-step response process
   - Blameless post-mortem template (full example with timeline, root cause, action items)

### Code Examples Required
1. Structured JSON logging configuration (logback-spring.xml)
2. Custom Micrometer metrics (Counter + Timer)
3. OpenTelemetry Spring Boot configuration (YAML)
4. Prometheus alerting rules (YAML)
5. Custom HealthIndicator (Spring Boot Actuator)

### Diagrams Required (minimum 5)
Three pillars of observability, Distributed trace tree, Log aggregation pipeline, SLI→SLO→SLA hierarchy, Incident response process flow, Error budget decision framework

### Real-World Case Study
Design complete observability strategy for an 8-service e-commerce platform: what to log, what metrics to collect, sampling strategy, dashboard layout, alerting rules, SLOs.

### Exercises (3)
1. Observability strategy design (logging, metrics, tracing, dashboards)
2. SLO definition for order service (3 SLIs, targets, error budgets, burn rate alerts)
3. Chaos experiment design (5 experiments for payment service)

### Self-Check Questions (7)
Cover: three pillars complementarity, structured JSON logging, trace/span, Golden Signals + RED, SLI/SLO/SLA + error budgets, chaos engineering experiments, post-mortem quality.

### References
Include: Google SRE Book, "Release It!" (Nygard), OpenTelemetry docs, Prometheus docs, Microsoft monitoring best practices.