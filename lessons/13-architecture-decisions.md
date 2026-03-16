# Lesson 13: Architecture Decision Records & Documentation

> **Prerequisites:** Lessons 01-12  
> **Time estimate:** 4-6 hours

---

## Table of Contents

1. [Architecture Decision Records (ADRs)](#1-adrs)
2. [C4 Model for Architecture Diagrams](#2-c4-model)
3. [Architecture Review Process](#3-review-process)
4. [Trade-Off Analysis](#4-trade-off-analysis)
5. [Exercises](#5-exercises)
6. [Self-Check Questions](#6-self-check-questions)
7. [Answers](#7-answers)

---

## 1. Architecture Decision Records (ADRs)

### Why ADRs?

```
6 months later: "Why did we choose Kafka instead of RabbitMQ?"
Without ADRs: Nobody remembers. New team members guess.
With ADRs: Read ADR-005, understand context, constraints, and reasoning.
```

### Template (Michael Nygard Format)

```markdown
# ADR-005: Use Apache Kafka for Event Streaming

## Status
Accepted (2024-03-15)

## Context
Our e-commerce platform needs asynchronous communication between 8 microservices.
We need: event ordering per entity, replay capability for new consumers,
high throughput (10K events/sec peak), and retention for 7 days.

## Decision
We will use Apache Kafka as our event streaming platform.

## Alternatives Considered
1. **RabbitMQ** — Rejected: No built-in replay, weaker ordering guarantees,
   lower throughput at scale.
2. **AWS SQS/SNS** — Rejected: No replay, no ordering guarantee,
   vendor lock-in concern.
3. **Azure Service Bus** — Rejected: Limited partition count, higher cost at
   our volume.

## Consequences
### Positive
- Message replay enables new consumers to process historical events
- Ordering per partition key (entity ID)
- Proven at scale (LinkedIn, Netflix)

### Negative
- Operational complexity (ZooKeeper/KRaft, topic management)
- Team needs Kafka expertise (training required)
- Higher infrastructure cost than RabbitMQ

### Risks
- Kafka consumer lag could cause processing delays under peak load
- Mitigation: KEDA autoscaling on consumer lag metric
```

### ADR Lifecycle

```
Proposed → Accepted → Superseded (by ADR-012)
                    → Deprecated
                    → Rejected
```

Store ADRs in the repository: `docs/adr/` — versioned with the code.

---

## 2. C4 Model for Architecture Diagrams

### Four Levels of Zoom

```
Level 1: System Context    — Your system + external actors
Level 2: Container         — High-level tech building blocks
Level 3: Component         — Inside a single container
Level 4: Code              — UML class diagrams (rarely needed)

Rule: Start at Level 1 for stakeholders, Level 2 for architects,
      Level 3 for developers. Skip Level 4.
```

### Level 1: System Context

```
┌─────────────┐         ┌─────────────────┐         ┌──────────┐
│  Customer   │────────→│   E-Commerce    │────────→│ Payment  │
│  (Browser/  │         │   Platform      │         │ Gateway  │
│   Mobile)   │←────────│                 │←────────│ (Stripe) │
└─────────────┘         └─────────────────┘         └──────────┘
                                │
                                ▼
                        ┌──────────────┐
                        │  Shipping    │
                        │  Provider    │
                        │  (DHL API)   │
                        └──────────────┘
```

### Level 2: Container

```
┌──────────────────── E-Commerce Platform ────────────────────┐
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Web App  │  │ Order    │  │ Product  │  │ Payment  │   │
│  │ (React)  │  │ Service  │  │ Service  │  │ Service  │   │
│  │          │  │ (Java)   │  │ (Java)   │  │ (Java)   │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │              │              │              │         │
│       └──────────────┼──────────────┼──────────────┘         │
│                      │              │                         │
│              ┌───────┴──┐    ┌─────┴──────┐                 │
│              │PostgreSQL │    │   Redis    │                 │
│              │          │    │   Cache    │                 │
│              └──────────┘    └────────────┘                 │
│                      │                                       │
│              ┌───────┴──┐                                    │
│              │  Kafka   │                                    │
│              └──────────┘                                    │
└──────────────────────────────────────────────────────────────┘
```

### Diagrams as Code

Use tools like Structurizr, PlantUML, or Mermaid to keep diagrams in version control:

```
# Mermaid (renders in GitHub markdown)
graph TD
    A[Web Browser] --> B[API Gateway]
    B --> C[Order Service]
    B --> D[Product Service]
    C --> E[(PostgreSQL)]
    C --> F[Kafka]
    D --> G[(Redis)]
```

---

## 3. Architecture Review Process

### Architecture Review Board (ARB)

```
When to trigger a review:
- New service or major component
- New technology adoption
- Cross-team API changes
- Security-sensitive changes
- Changes affecting SLOs

Review checklist:
☐ ADR written with alternatives considered
☐ C4 diagrams (Level 1 and 2 minimum)
☐ Non-functional requirements addressed (scalability, security, reliability)
☐ API contract defined (OpenAPI/protobuf)
☐ Data model and ownership clear
☐ Deployment and rollback strategy
☐ Monitoring and alerting plan
☐ Cost estimate
```

### Fitness Functions

Automated checks that verify architectural properties:

```
Examples:
- No cyclic dependencies between services
- All APIs have OpenAPI specs
- All services have health endpoints
- No service directly accesses another's database
- All Docker images are scanned
- Response time < 200ms for 95th percentile
```

---

## 4. Trade-Off Analysis

### Architecture Trade-Off Analysis Method (ATAM)

For every decision, explicitly state the trade-offs:

```
Decision: Use event sourcing for the order domain

Quality Attribute | Impact
Auditability      | ++ (complete history of every state change)
Scalability       | +  (append-only writes scale well)
Complexity        | -- (event replay, snapshots, projections)
Query performance | -  (need separate read models / CQRS)
Consistency       | -  (eventual consistency between write and read models)

Conclusion: Worth it for the order domain (audit requirements).
            Not worth it for the product catalog (simple CRUD).
```

---

## 5. Exercises

### Exercise 1: Write ADRs

Write ADRs for these decisions:
1. Choosing between PostgreSQL and MongoDB for the order service
2. Adopting a service mesh (Istio) vs. library-based resilience (Resilience4j)
3. Choosing between REST and gRPC for internal service communication

### Exercise 2: C4 Diagrams

Create C4 Level 1 and Level 2 diagrams for a food delivery platform with: customer app, restaurant app, driver app, order management, payment, real-time tracking, notification service.

### Exercise 3: Trade-Off Analysis

Perform ATAM analysis for: monolith vs. microservices for a startup MVP. Consider: time-to-market, scalability, operational complexity, team size, cost.

---

## 6. Self-Check Questions

1. What is an ADR and why should architects write them?
2. Describe the four levels of the C4 model.
3. What should trigger an architecture review?
4. How do fitness functions help maintain architecture quality?

---

## 7. Answers

### Answer 1
An ADR documents a significant architecture decision: the context (why), the decision (what), alternatives considered (what else), and consequences (trade-offs). Architects write them because: decisions are forgotten over time, new team members need context, it forces explicit trade-off thinking, it prevents revisiting settled decisions, and it creates an audit trail of architectural evolution.

### Answer 2
**Level 1 (System Context):** Shows your system as a single box with external actors (users, external systems). For stakeholder communication. **Level 2 (Container):** Shows the high-level building blocks inside your system (services, databases, message brokers). For technical discussions. **Level 3 (Component):** Shows the internal components of a single container (controllers, services, repositories). For development teams. **Level 4 (Code):** Class-level diagrams. Rarely used — code is the source of truth at this level.

### Answer 3
Trigger reviews for: new services or major components, new technology adoption, cross-team API changes, security-sensitive changes, changes affecting SLOs, significant cost increases, changes to data ownership boundaries.

### Answer 4
Fitness functions are automated tests that verify architectural properties continuously. Examples: no cyclic dependencies, all APIs documented, all services have health checks, no direct database access across service boundaries. They run in CI/CD and prevent architectural drift — the gradual erosion of design principles as teams make expedient shortcuts.

---

*Previous: [12 - Scalability & Performance](12-scalability-performance.md)*  
*Next: [14 - Cost Optimization & FinOps](14-cost-optimization.md)*