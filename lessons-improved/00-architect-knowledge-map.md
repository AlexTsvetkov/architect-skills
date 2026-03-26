# Lesson 00: Cloud-Native Architect Knowledge Map

> **Created:** March 2026  
> **Last updated:** March 2026  
> **Purpose:** Course entry point and navigation hub

---

## Course Introduction

### Who This Is For

This course is designed for **experienced Java developers and team leads** (5+ years) transitioning to cloud-native architecture roles. You are the ideal reader if you:

- Have strong **Java / Spring Boot** experience in enterprise environments
- Have built or maintained **monolithic applications** (Java EE, Spring MVC, large WAR/EAR deployments)
- Have read **"System Design Interview"** by Alex Xu — you understand high-level system design
- Have partially read **"Designing Data-Intensive Applications"** by Martin Kleppmann — you have foundations in data systems
- Want a **structured, deep, practical** path from senior developer to software architect

### What You'll Learn

By completing this course, you will be able to:

1. **Evaluate and select** architecture styles for different business contexts
2. **Design microservices** using Domain-Driven Design and proper decomposition strategies
3. **Architect distributed systems** with appropriate consistency, replication, and partitioning strategies
4. **Deploy and operate** containerized applications on Kubernetes
5. **Apply resilience patterns** (circuit breakers, retries, bulkheads) to production systems
6. **Design APIs** using REST, gRPC, and GraphQL with proper versioning and security
7. **Implement observability** with metrics, logs, distributed traces, SLOs, and error budgets
8. **Secure cloud-native systems** with zero-trust architecture, OAuth 2.0, and supply chain security
9. **Build event-driven architectures** with Kafka, schema evolution, and reliable messaging patterns
10. **Automate delivery** with CI/CD pipelines, GitOps, and progressive deployment strategies
11. **Optimize performance and cost** with auto-scaling, caching, FinOps, and capacity planning
12. **Document and communicate** architecture decisions using ADRs, C4 model, and fitness functions

### How to Use This Guide

```
┌─────────────────────────────────────────────────────────┐
│                    LEARNING APPROACH                      │
├──────────────┬──────────────────────────────────────────┤
│ Each lesson  │ • Concept explanation with diagrams       │
│ contains:    │ • Java/Spring Boot code examples          │
│              │ • Hands-on exercises                      │
│              │ • Self-check questions with answers        │
├──────────────┼──────────────────────────────────────────┤
│ Recommended  │ • 2 weeks per lesson                      │
│ pace:        │ • Read the lesson (2-3 hours)             │
│              │ • Complete exercises (2-4 hours)           │
│              │ • Parallel reading from book list          │
├──────────────┼──────────────────────────────────────────┤
│ Your         │ • ✅ marks topics you already know        │
│ baseline:    │ • Focus time on 🔲 topics                 │
│              │ • Use exercises to validate understanding  │
└──────────────┴──────────────────────────────────────────┘
```

---

## Knowledge Baseline Assessment

Map your existing knowledge against the course topics. This helps you prioritize where to spend more time.

| Area | Status | Source | Course Lessons |
|------|--------|--------|----------------|
| Java ecosystem (Spring Boot, JPA, etc.) | ✅ Strong | Professional experience | All lessons use Java examples |
| Enterprise monolith architecture | ✅ Strong | Professional experience | 01, 02, 03 (migration context) |
| System design fundamentals | ✅ Read | Alex Xu — System Design Interview | 04, 12 (extends your knowledge) |
| Data-intensive applications | 🟡 Partial | Kleppmann DDIA (half read) | 04, 10 (fills gaps) |
| REST API design | 🟡 Partial | Professional experience | 07 (deepens to expert level) |
| CI/CD basics | 🟡 Partial | Professional experience | 11 (extends to GitOps) |
| Kubernetes / Containers | 🔲 To learn | — | 05 (from scratch) |
| Cloud-native patterns | 🔲 To learn | — | 06 (comprehensive catalog) |
| Service mesh / Observability | 🔲 To learn | — | 08 (full stack) |
| Event-driven architecture | 🔲 To learn | — | 10 (Kafka deep dive) |
| Security (Zero Trust, OAuth 2.0) | 🔲 To learn | — | 09 (cloud-native security) |
| Architecture documentation (ADRs, C4) | 🔲 To learn | — | 13 (architect practices) |
| Cloud cost optimization / FinOps | 🔲 To learn | — | 14 (practical FinOps) |

---

## Course Structure

The 15 lessons are organized into 6 thematic parts that build on each other:

```
                        COURSE STRUCTURE
    ┌──────────────────────────────────────────────┐
    │           Part I: FOUNDATIONS (01-03)          │
    │  Cloud-Native │ Architecture │ Microservices   │
    │  Fundamentals │ Styles       │ Deep Dive       │
    ├──────────────────────────────────────────────┤
    │     Part II: DISTRIBUTED SYSTEMS (04, 10)     │
    │  Distributed Systems  │  Event-Driven          │
    │  & Data               │  Architecture          │
    ├──────────────────────────────────────────────┤
    │    Part III: CLOUD INFRASTRUCTURE (05, 06, 08)│
    │  Containers &  │ Cloud-Native │ Observability  │
    │  Orchestration │ Patterns     │ & Reliability  │
    ├──────────────────────────────────────────────┤
    │      Part IV: SECURITY & APIs (07, 09)        │
    │  API Design &       │  Security in             │
    │  Management          │  Cloud-Native Systems    │
    ├──────────────────────────────────────────────┤
    │    Part V: DELIVERY & OPERATIONS (11, 12)     │
    │  CI/CD &            │  Scalability &           │
    │  GitOps             │  Performance             │
    ├──────────────────────────────────────────────┤
    │   Part VI: ARCHITECTURE PRACTICE (13, 14)     │
    │  Architecture       │  Cost Optimization       │
    │  Decisions & Docs   │  & FinOps                │
    └──────────────────────────────────────────────┘
```

---

## Part I — Foundations of Software Architecture

### Lesson 01: Cloud-Native Architecture Fundamentals
`lessons/01-cloud-native-fundamentals.md`

**Key topics covered:**
- What "cloud-native" really means — CNCF definition vs. practical reality
- The four pillars: microservices, containers, CI/CD, DevOps
- Key principles: design for failure, cattle not pets, immutable infrastructure
- The 12-Factor App methodology — all 12 factors with Java/Spring Boot code examples
- Beyond 12-Factor: the 15-factor app (API First, Telemetry, Auth)
- Cloud-Native Maturity Model (5 levels: Cloud Hostile → Cloud Native + Serverless)
- The CNCF landscape — curated map of graduated projects
- Evolution: Monolith → SOA → Microservices → Serverless
- Cloud-Native vs. Cloud-Hosted vs. Cloud-Ready — definitions and migration strategy
- Docker Compose for dev/prod parity

**Connection to your experience:** Your enterprise monolith background gives you the perfect "before picture." Each 12-Factor principle maps directly to a pain point you've likely experienced — hardcoded configs, stateful servers, manual deployments. This lesson bridges your experience to cloud-native thinking.

**Key exercises:**
- 12-Factor Audit of a Java application
- Cloud-Native Maturity Assessment
- Decision framework: When NOT to go cloud-native
- Hands-on: Build a 12-Factor Spring Boot App

---

### Lesson 02: Architecture Styles & Patterns
`lessons/02-architecture-styles.md`

**Key topics covered:**
- Architecture characteristics (quality attributes) and tradeoffs
- The Laws of Software Architecture: "Everything is a tradeoff"
- N-Tier / Layered architecture — with Spring Boot project structure
- Microservices architecture — key characteristics and the "microservices premium"
- Event-Driven Architecture — pub/sub vs. event streaming
- CQRS and Event Sourcing — separate read/write models
- Web-Queue-Worker pattern
- Big Data architectures — Lambda vs. Kappa
- Choreography vs. Orchestration — Saga pattern introduction
- Space-Based Architecture
- Micro-Frontends — vertical team ownership
- Modular Monolith — with ArchUnit boundary enforcement
- Architecture style decision framework and comparison matrix

**Connection to your experience:** Your N-tier monolith is one of these architecture styles. Understanding the full spectrum — from layered to microservices to event-driven — helps you see where your current systems sit and where they could evolve. The modular monolith pattern is often the best first step.

**Key exercises:**
- Architecture Style Mapping — match systems to styles
- Architecture Tradeoff Analysis — monolith-to-microservices migration proposal
- CQRS Decision — compare CQRS and non-CQRS approaches
- Event-Driven Design — order processing system

---

### Lesson 03: Microservices Architecture — Deep Dive
`lessons/03-microservices-deep-dive.md`

**Key topics covered:**
- Domain-Driven Design (DDD) — bounded contexts, ubiquitous language, context maps
- Tactical DDD — aggregates, entities, value objects with Java code examples
- Decomposition strategies: by business capability, by subdomain, strangler fig
- Data ownership — database-per-service pattern and cross-service query solutions
- Inter-service communication: REST vs. gRPC (sync), commands vs. events (async)
- The synchronous chain problem (availability math: 99.9%^5 = 99.5%)
- API Gateway and Backend for Frontend (BFF) patterns
- Service discovery — client-side vs. server-side, Kubernetes built-in
- Distributed transactions — Saga pattern (orchestration vs. choreography)
- Testing: contract testing (Pact), integration testing (Testcontainers), chaos engineering
- Anti-patterns: distributed monolith, nano-services, shared database

**Connection to your experience:** Decomposing your enterprise monolith is the central challenge. DDD's bounded contexts map directly to your existing domain modules. The strangler fig pattern lets you migrate incrementally rather than doing a risky big-bang rewrite.

**Key exercises:**
- Domain Decomposition — identify bounded contexts, draw context map
- Saga Design — hotel booking system with failure handling
- Data Ownership Challenge — break monolith JOIN queries
- Monolith Decomposition — full plan for a 500K LOC e-commerce monolith

---

## Part II — Distributed Systems & Data

### Lesson 04: Distributed Systems & Data Management
`lessons/04-distributed-systems-data.md`

**Key topics covered:**
- CAP theorem — practical implications (why "pick two" is misleading)
- PACELC theorem — more useful than CAP
- Consistency models: strong, eventual, causal, read-your-writes
- Distributed consensus: Raft algorithm conceptual walkthrough
- Replication strategies: leader-follower, multi-leader, leaderless (Dynamo-style)
- Partitioning/sharding: key-range, hash, consistent hashing
- Distributed caching: cache-aside, read-through, write-through, write-behind
- Redis in cloud-native architecture — Kubernetes deployment
- CQRS implementation with Spring Boot
- Event sourcing: event store design, snapshots, schema evolution
- Change Data Capture (CDC) with Debezium
- Polyglot persistence — database decision guide

**Connection to your experience:** You've read half of DDIA — this lesson fills the gaps and extends to practical implementation. The concepts of replication and partitioning connect to the scaling challenges you've seen in enterprise databases. CQRS and CDC patterns solve the "how do I break the monolith's shared database" problem.

**Key exercises:**
- Consistency model design for e-commerce features
- Sharding strategy for 500M user records
- Caching architecture for product catalog
- CQRS for order management system

---

### Lesson 10: Event-Driven Architecture & Messaging
`lessons/10-event-driven-architecture.md`

**Key topics covered:**
- Event-driven patterns: event notification, event-carried state transfer, event sourcing
- Message brokers comparison: Kafka, RabbitMQ, AWS SQS/SNS, Azure Service Bus
- Apache Kafka deep dive: partitions, consumer groups, exactly-once semantics
- Spring Boot Kafka producer and consumer code
- Event schema evolution — backward/forward/full compatibility
- Schema Registry workflow with Avro
- Transactional outbox pattern — solving the dual-write problem
- Dead letter queues and poison message handling
- Idempotent consumers with duplicate detection
- Stream processing: Kafka Streams vs. Apache Flink

**Connection to your experience:** Enterprise messaging (JMS, MQ) gives you a foundation. Kafka-based architecture is fundamentally different — it's a distributed commit log, not a message queue. Understanding partitions, consumer groups, and exactly-once semantics is critical for building reliable event-driven systems.

**Key exercises:**
- Event design for order flow (10 events, schemas, topics, partition keys)
- Failure handling strategy (consumer failure, poison messages, idempotency)
- Schema evolution migration plan

---

## Part III — Cloud Infrastructure & Operations

### Lesson 05: Containers & Orchestration
`lessons/05-containers-orchestration.md`

**Key topics covered:**
- Docker deep dive: images, layers, caching, multi-stage builds for Java
- Docker security best practices (non-root, minimal base images)
- Docker Compose for local development (app, PostgreSQL, Redis, Kafka)
- Kubernetes architecture: control plane, worker nodes, pods
- Kubernetes core objects: Deployments, Services, ConfigMaps, Secrets, Ingress
- Advanced workloads: StatefulSets, DaemonSets, Jobs, CronJobs
- Kubernetes networking: Services, Network Policies, Service Mesh
- Persistent storage: PV, PVC, StorageClasses, access modes
- Resource management: requests, limits, QoS classes
- Helm charts: structure, templating, values, install/upgrade/rollback
- HPA (Horizontal Pod Autoscaler) configuration
- Managed Kubernetes comparison: AKS vs. EKS vs. GKE

**Connection to your experience:** You deploy WAR files to application servers. Kubernetes is the cloud-native equivalent — but instead of managing servers, you declare desired state and the platform handles the rest. Your Spring Boot fat JAR maps naturally to a Docker container.

**Key exercises:**
- Containerize Spring Boot app (Dockerfile, docker-compose, <300MB image)
- Deploy to Kubernetes (7 manifests)
- Create Helm chart
- Design K8s architecture for 8-service e-commerce platform

---

### Lesson 06: Cloud-Native Design Patterns
`lessons/06-cloud-native-patterns.md`

**Key topics covered:**
- Circuit Breaker — state machine, Resilience4j implementation with fallback
- Retry with Exponential Backoff and Jitter — why jitter prevents thundering herd
- Bulkhead — thread pool isolation with Resilience4j
- Timeout — TimeLimiter with CompletableFuture
- Health Endpoint Monitoring — custom HealthIndicator, K8s integration
- Rate Limiting / Throttling — RateLimiter with 429 fallback
- Sidecar Pattern — Pod diagram with Envoy, Fluentd, Vault Agent examples
- Ambassador and Adapter patterns
- Strangler Fig Pattern — 3-phase migration with implementation approaches
- Branch by Abstraction — interface + feature flag code example
- Anti-Corruption Layer — model translation between new and legacy
- Gateway Aggregation and Competing Consumers
- Transactional Outbox — dual-write problem and solution
- Valet Key Pattern — pre-signed URL flow

**Connection to your experience:** The patterns in "Release It!" (Nygard) — circuit breakers, bulkheads, timeouts — are the resilience patterns you need when moving from a monolith (where failures are local) to microservices (where failures are distributed). The strangler fig pattern is your migration strategy.

**Key exercises:**
- Resilience design for payment service
- Migration plan for 500K LOC monolith
- Pattern combination for food delivery platform

---

### Lesson 08: Observability & Reliability Engineering
`lessons/08-observability-reliability.md`

**Key topics covered:**
- Monitoring vs. Observability distinction
- Three pillars: structured logs, metrics, distributed traces
- Structured JSON logging with logback-spring.xml
- Log aggregation pipeline: App → Fluentd → Elasticsearch → Kibana
- Distributed tracing with OpenTelemetry and Spring Boot
- W3C traceparent header propagation
- Four Golden Signals (Google SRE): latency, traffic, errors, saturation
- RED Method (services) and USE Method (resources)
- Prometheus + Grafana: custom Micrometer metrics
- Prometheus alerting rules
- SLIs, SLOs, SLAs — definitions, error budgets, burn rate alerts
- Chaos engineering: 5 principles, experiments, Game Days
- Incident management: 5-step response process
- Blameless post-mortem template

**Connection to your experience:** In a monolith, you debug by reading one log file. In microservices, a single user request may traverse 10 services. Distributed tracing, structured logging with correlation IDs, and golden signals are how you maintain visibility across a distributed system.

**Key exercises:**
- Observability strategy design for 8-service platform
- SLO definition for order service (SLIs, targets, error budgets)
- Chaos experiment design for payment service

---

## Part IV — Security, APIs, and Cross-Cutting Concerns

### Lesson 07: API Design & Management
`lessons/07-api-design.md`

**Key topics covered:**
- REST API design: Richardson Maturity Model, resource naming, filtering, pagination
- Error responses: RFC 7807 (Problem Details) format
- HTTP status codes — 12 essential codes with usage guidance
- Pagination: offset-based vs. cursor-based with examples
- Idempotency: safe methods, Idempotency-Key header for POST
- gRPC: Protocol Buffers, communication patterns, when to use vs. REST
- GraphQL: query examples, federation for microservices, when to use/avoid
- API versioning: URL path, header, query param — comparison
- API security: OAuth 2.0, OpenID Connect, JWT validation
- API Gateway patterns: routing, offloading, aggregation
- Consumer-driven contract testing with Pact
- OpenAPI/Swagger specification

**Connection to your experience:** You've built REST APIs with Spring MVC. This lesson elevates your API design to architect level — versioning strategies, contract testing, choosing between REST/gRPC/GraphQL, and designing API gateway configurations for microservices.

**Key exercises:**
- Design REST API for library management (15 endpoints, RFC 7807, pagination)
- Versioning strategy for a breaking change
- gRPC vs REST decision for 5 scenarios

---

### Lesson 09: Security in Cloud-Native Systems
`lessons/09-security.md`

**Key topics covered:**
- Zero-trust architecture vs. traditional perimeter security
- Identity and Access Management: RBAC vs. ABAC vs. ReBAC
- OAuth 2.0 / OpenID Connect: Authorization Code flow, Client Credentials flow
- JWT structure, validation steps, stateless authentication
- Token architecture in microservices (Gateway → Service → Service)
- Service-to-service authentication: mTLS with service mesh, JWT propagation
- Secret management: Kubernetes Secrets, HashiCorp Vault, cloud-native options
- Container security: image scanning, Pod Security Context, Pod Security Standards
- Network security: Network Policies, defense in depth (5 layers)
- Supply chain security: SBOM, image signing (cosign), admission controllers

**Connection to your experience:** Enterprise Java security (Spring Security, JAAS) protects a single application. Cloud-native security protects a distributed system — every service-to-service call must be authenticated, every container must be hardened, and secrets must be managed centrally. Zero-trust replaces the castle-and-moat model.

**Key exercises:**
- Security architecture for multi-tenant SaaS platform
- STRIDE threat model for payment system

---

## Part V — Delivery & Operations

### Lesson 11: CI/CD & GitOps
`lessons/11-cicd-gitops.md`

**Key topics covered:**
- CI/CD pipeline design: stages from commit to deployment
- Microservices CI/CD: independent pipelines per service
- Complete GitHub Actions pipeline YAML
- Trunk-based development vs. GitFlow comparison
- GitOps principles: ArgoCD architecture, Git repository structure
- Infrastructure as Code: Terraform example (AKS cluster + PostgreSQL)
- Deployment strategies: rolling update, blue-green, canary, A/B testing
- Feature flags: decouple deployment from release, Java implementation
- Pipeline security (shift-left): SAST, SCA, image scanning, DAST
- Environment management: dev, staging, production parity

**Connection to your experience:** Enterprise Java typically has long release cycles (monthly/quarterly) with manual deployment processes. CI/CD and GitOps automate the entire pipeline — from code commit to production deployment — enabling multiple deployments per day with confidence.

**Key exercises:**
- CI/CD pipeline design for 8 microservices
- Deployment strategy for payment service API change
- GitOps repository structure for 8 services, 3 environments

---

### Lesson 12: Scalability & Performance
`lessons/12-scalability-performance.md`

**Key topics covered:**
- Horizontal vs. vertical scaling — when to use each
- Auto-scaling: HPA, VPA, KEDA (event-driven), cluster autoscaler
- Load balancing: L4 vs. L7, algorithms (round robin, least connections, consistent hashing)
- CDN and edge computing: cache-control headers, static vs. dynamic content
- Caching strategies: cache-aside, read-through, write-through, write-behind
- Database scaling: read replicas (Spring Boot config), sharding, connection pooling (HikariCP)
- Asynchronous processing: work queues, CompletableFuture, Java 21 virtual threads, Spring @Async
- Performance testing: load, stress, soak testing with JMeter/Gatling/k6
- Capacity planning: Little's Law (L = λW), back-of-the-envelope calculations
- Back pressure and flow control: Reactive Streams, rate limiting, queue depth monitoring

**Connection to your experience:** Alex Xu's system design concepts come alive here. You'll learn to actually implement the scaling strategies you read about — auto-scaling on Kubernetes, caching with Redis, database read replicas, and capacity planning with real math.

**Key exercises:**
- Auto-scaling design for e-commerce platform
- Caching strategy for product catalog
- Database scaling for order service
- Black Friday preparation (capacity planning, performance testing)

---

## Part VI — Architecture Practice

### Lesson 13: Architecture Decision Records & Documentation
`lessons/13-architecture-decisions.md`

**Key topics covered:**
- Architecture Decision Records (ADRs) — Michael Nygard template
- Complete ADR example: Kafka vs. RabbitMQ for event streaming
- ADR lifecycle: Proposed → Accepted → Superseded/Deprecated/Rejected
- C4 Model: System Context, Container, Component, Code (4 zoom levels)
- Diagrams as Code: Structurizr, PlantUML, Mermaid (GitHub-renderable)
- Architecture review process: triggers, checklist, Architecture Review Board
- Fitness functions — automated architecture verification with ArchUnit
- Trade-off analysis (ATAM) — quality attribute impact tables
- Technical debt management — Martin Fowler's quadrant
- Communicating architecture to different audiences (executives, developers, ops)

**Connection to your experience:** As a senior developer, you made technical decisions informally. As an architect, you need to document decisions (ADRs), communicate architecture visually (C4), and enforce architectural properties automatically (fitness functions). This lesson teaches the "soft skills" of architecture.

**Key exercises:**
- Write 3 ADRs for technology choices
- Create C4 diagrams for food delivery platform
- ATAM analysis: monolith vs. microservices for startup MVP

---

### Lesson 14: Cost Optimization & FinOps
`lessons/14-cost-optimization.md`

**Key topics covered:**
- Cloud cost model: compute, storage, database, networking breakdown
- Pay-per-use vs. Reserved vs. Spot/Preemptible vs. Savings Plans
- Right-sizing: the over-provisioning problem (80-90% typical waste)
- Kubernetes resource optimization: before/after YAML
- Spot/preemptible instances: savings, use cases, mixed strategy
- Serverless economics: break-even analysis (when serverless < containers < VMs)
- FinOps framework: cost allocation tagging, weekly reviews, anomaly detection
- Architecture cost patterns: 6 decisions and their cost impact
- Multi-cloud and cloud-agnostic considerations: vendor lock-in spectrum
- TCO analysis: cloud vs. on-premises, hidden costs

**Connection to your experience:** On-premises infrastructure has hidden costs (hardware, data center, ops staff). Cloud makes costs visible but also unpredictable. FinOps practices and right-sizing techniques help you architect systems that are not just scalable and resilient, but also cost-efficient.

**Key exercises:**
- Cost analysis and optimization for 8-service platform ($15K/month → 40% reduction)
- Build vs. buy analysis (managed Kafka vs. self-hosted)

---

## Recommended Reading Path

### 📚 Must-Read (Priority Order)

| # | Book | Why | When |
|---|------|-----|------|
| 1 | **"Fundamentals of Software Architecture"** — Mark Richards & Neal Ford | The best modern overview of architecture styles, patterns, and the architect role. Directly relevant to everything in this curriculum. | Alongside Lessons 01-02 |
| 2 | **"Building Microservices" (2nd Ed.)** — Sam Newman | The definitive guide to microservices. Covers decomposition, communication, deployment, and organizational aspects. | Alongside Lessons 03-06 |
| 3 | **Finish "Designing Data-Intensive Applications"** — Martin Kleppmann | You've read half — the second half covers distributed consensus, batch/stream processing, and future of data systems. Essential for Lessons 04 and 10. | Alongside Lesson 04 |
| 4 | **"Cloud Native Patterns"** — Cornelia Davis | Practical patterns for building cloud-native applications. Bridges theory and implementation. | Alongside Lesson 06 |
| 5 | **"Software Architecture: The Hard Parts"** — Neal Ford, Mark Richards, Pramod Sadalage, Zhamak Dehghani | Tackles difficult tradeoffs in distributed architectures. Data decomposition, service granularity, contracts. | After Lessons 03-04 |

### 📖 Highly Recommended

| # | Book | Why | When |
|---|------|-----|------|
| 6 | **"Domain-Driven Design Distilled"** — Vaughn Vernon | Concise intro to DDD — critical for microservices decomposition. Follow with Evans' book for more depth. | Alongside Lesson 03 |
| 7 | **"Release It!" (2nd Ed.)** — Michael Nygard | Stability patterns and anti-patterns for production systems. Circuit breakers, bulkheads, timeouts. | Alongside Lessons 06, 08 |
| 8 | **"Kubernetes in Action" (2nd Ed.)** — Marko Lukša | Comprehensive Kubernetes guide. Better than most online tutorials for deep understanding. | Alongside Lesson 05 |
| 9 | **"Team Topologies"** — Matthew Skelton & Manuel Pais | How to organize teams for fast flow. Conway's Law applied. Architecture and team structure are inseparable. | After Lesson 03 |
| 10 | **"The Phoenix Project" / "The Unicorn Project"** — Gene Kim | Novel format — makes DevOps and flow principles tangible. Good mental model for CI/CD and organizational change. | Alongside Lesson 11 |

### 🔬 Deep Dives (When You Want More)

| # | Book | Why |
|---|------|-----|
| 11 | **"Designing Distributed Systems"** — Brendan Burns | Patterns for distributed systems by Kubernetes co-creator. Sidecar, ambassador, adapter patterns. |
| 12 | **"Production-Ready Microservices"** — Susan Fowler | Operational excellence checklist for microservices. |
| 13 | **"Monolith to Microservices"** — Sam Newman | Practical migration patterns. Strangler fig, branch by abstraction. |
| 14 | **"Implementing Domain-Driven Design"** — Vaughn Vernon | Deep DDD implementation. For when "DDD Distilled" isn't enough. |
| 15 | **"System Design Interview Vol 2"** — Alex Xu | You've read Vol 1. Vol 2 covers advanced topics: proximity service, nearby friends, stock exchange. |

---

## Online Resources

| Resource | URL | Best For |
|----------|-----|----------|
| Microsoft Azure Architecture Center | https://learn.microsoft.com/en-us/azure/architecture/ | Architecture styles, patterns, best practices |
| CNCF Cloud Native Interactive Landscape | https://landscape.cncf.io/ | Understanding the ecosystem |
| Martin Fowler's Architecture Articles | https://martinfowler.com/architecture/ | Timeless architecture articles |
| 12factor.net | https://12factor.net/ | The 12-Factor App methodology |
| microservices.io | https://microservices.io/ | Microservices patterns catalog |
| The Architect Elevator (Gregor Hohpe) | https://architectelevator.com/ | Architecture insights and strategy |
| InfoQ Architecture & Design | https://www.infoq.com/architecture-design/ | Current architecture trends |
| ByteByteGo Newsletter (Alex Xu) | https://blog.bytebytego.com/ | System design content (continuation of his books) |
| Google SRE Books | https://sre.google/books/ | Site Reliability Engineering practices |
| C4 Model | https://c4model.com/ | Architecture diagramming standard |

---

## Certifications to Consider

| Certification | Provider | Relevance | Difficulty |
|---------------|----------|-----------|------------|
| AWS Solutions Architect (Associate → Professional) | AWS | Most recognized cloud architecture certification | Medium → Hard |
| Azure Solutions Architect Expert (AZ-305) | Microsoft | Strong match for Azure-focused organizations | Medium |
| CKA (Certified Kubernetes Administrator) | CNCF | Validates Kubernetes operational knowledge | Medium |
| CKAD (Certified Kubernetes Application Developer) | CNCF | Validates Kubernetes app development skills | Medium |
| iSAQB CPSA-F / CPSA-A | iSAQB | International software architecture certification (vendor-neutral) | Medium → Hard |

---

## Learning Schedule (28 Weeks)

| Weeks | Lesson | Part | Parallel Reading |
|-------|--------|------|-----------------|
| 1-2 | **01:** Cloud-Native Fundamentals | Part I: Foundations | "Fundamentals of Software Architecture" (Part I) |
| 3-4 | **02:** Architecture Styles & Patterns | Part I: Foundations | "Fundamentals of Software Architecture" (Part II) |
| 5-6 | **03:** Microservices Deep Dive | Part I: Foundations | "Building Microservices" (Ch 1-6) + "DDD Distilled" |
| 7-8 | **04:** Distributed Systems & Data | Part II: Distributed Systems | Finish DDIA (remaining chapters) |
| 9-10 | **05:** Containers & Orchestration | Part III: Cloud Infra | "Kubernetes in Action" (Part I) |
| 11-12 | **06:** Cloud-Native Patterns | Part III: Cloud Infra | "Cloud Native Patterns" + "Release It!" |
| 13-14 | **07:** API Design & Management | Part IV: Security & APIs | "Building Microservices" (Ch 7-10) |
| 15-16 | **08:** Observability & Reliability | Part III: Cloud Infra | "Release It!" (Part II) + Google SRE Book |
| 17-18 | **09:** Security | Part IV: Security & APIs | OWASP Cloud-Native Security Guide |
| 19-20 | **10:** Event-Driven Architecture | Part II: Distributed Systems | DDIA (Stream processing chapters) |
| 21-22 | **11:** CI/CD & GitOps | Part V: Delivery & Ops | "The Phoenix Project" |
| 23-24 | **12:** Scalability & Performance | Part V: Delivery & Ops | "System Design Interview Vol 2" |
| 25-26 | **13:** Architecture Decisions & Docs | Part VI: Architecture Practice | "Software Architecture: The Hard Parts" |
| 27-28 | **14:** Cost Optimization & FinOps | Part VI: Architecture Practice | "Team Topologies" |

---

## Quick Navigation

| Lesson | File | Time Est. |
|--------|------|-----------|
| 00 - Knowledge Map | `lessons/00-architect-knowledge-map.md` | 1 hour |
| 01 - Cloud-Native Fundamentals | `lessons/01-cloud-native-fundamentals.md` | 4-6 hours |
| 02 - Architecture Styles | `lessons/02-architecture-styles.md` | 4-6 hours |
| 03 - Microservices Deep Dive | `lessons/03-microservices-deep-dive.md` | 6-8 hours |
| 04 - Distributed Systems & Data | `lessons/04-distributed-systems-data.md` | 6-8 hours |
| 05 - Containers & Orchestration | `lessons/05-containers-orchestration.md` | 6-8 hours |
| 06 - Cloud-Native Patterns | `lessons/06-cloud-native-patterns.md` | 4-6 hours |
| 07 - API Design & Management | `lessons/07-api-design.md` | 4-6 hours |
| 08 - Observability & Reliability | `lessons/08-observability-reliability.md` | 4-6 hours |
| 09 - Security | `lessons/09-security.md` | 4-6 hours |
| 10 - Event-Driven Architecture | `lessons/10-event-driven-architecture.md` | 4-6 hours |
| 11 - CI/CD & GitOps | `lessons/11-cicd-gitops.md` | 4-6 hours |
| 12 - Scalability & Performance | `lessons/12-scalability-performance.md` | 4-6 hours |
| 13 - Architecture Decisions | `lessons/13-architecture-decisions.md` | 3-4 hours |
| 14 - Cost Optimization | `lessons/14-cost-optimization.md` | 3-4 hours |

**Total estimated time:** ~70-90 hours of study + exercises

---

*Created: March 2026*  
*Last updated: March 2026*

*Next lesson: [01 — Cloud-Native Architecture Fundamentals](01-cloud-native-fundamentals.md)*