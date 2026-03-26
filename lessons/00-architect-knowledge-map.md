# Cloud-Native Architect Knowledge Map

> **For:** Experienced Java Developer / Team Lead transitioning to Cloud-Native Architecture  
> **Background assumed:** Java ecosystem, enterprise application development, System Design (Alex Xu), partial DDIA (Kleppmann)  
> **Goal:** Systematic mastery of cloud-native architecture principles and practices

---

## How to Use This Guide

Each topic below maps to a dedicated lesson file (`01-xxx.md`, `02-xxx.md`, etc.) containing:
- **Concept explanation** with diagrams and real-world context
- **Exercises** — hands-on design tasks
- **Self-check questions** — test your understanding
- **Answers** — detailed solutions and reasoning

Lessons are ordered from foundational → advanced. Your existing knowledge (marked ✅) lets you move faster through some areas.

---

## Your Knowledge Baseline

| Area | Status | Source |
|------|--------|--------|
| Java ecosystem (Spring Boot, JPA, etc.) | ✅ Strong | Professional experience |
| Enterprise monolith architecture | ✅ Strong | Professional experience |
| System design fundamentals | ✅ Read | Alex Xu — System Design Interview |
| Data-intensive applications | 🟡 Partial | Kleppmann DDIA (half read) |
| Kubernetes / Containers | 🔲 To learn | — |
| Cloud-native patterns | 🔲 To learn | — |
| Service mesh / Observability | 🔲 To learn | — |

---

## Part I — Foundations of Software Architecture

### Lesson 01: Cloud-Native Architecture Fundamentals
`lessons/01-cloud-native-fundamentals.md`
- What "cloud-native" really means (CNCF definition vs. reality)
- 12-Factor App methodology — deep dive with Java/Spring examples
- Beyond 12-Factor: the 15-factor app
- Cloud-native vs. cloud-hosted vs. cloud-ready
- The CNCF landscape and ecosystem
- Comparing monolith → SOA → microservices → serverless evolution
- **Connection to your experience:** How enterprise monoliths map to cloud-native migration

### Lesson 02: Architecture Styles & Patterns
`lessons/02-architecture-styles.md`
- N-tier / layered architecture
- Microservices architecture
- Event-driven architecture
- CQRS and Event Sourcing
- Big data / lambda / kappa architectures
- Web-queue-worker
- Choreography vs. orchestration (Saga patterns)
- Space-based architecture
- Micro-frontends
- Modular monolith
- **Reference:** Microsoft Architecture Styles Guide mapping
- **Connection to your experience:** Monolith service layer → microservices decomposition

### Lesson 03: Microservices Architecture — Deep Dive
`lessons/03-microservices-deep-dive.md`
- Domain-Driven Design (DDD) — bounded contexts, aggregates, context maps
- Decomposition strategies: by business capability, by subdomain
- Data ownership and the database-per-service pattern
- Inter-service communication: sync (REST, gRPC) vs. async (events, messaging)
- API gateway patterns
- Service discovery
- Distributed transactions — Saga pattern (choreography vs. orchestration)
- Testing strategies: contract testing, integration testing, chaos engineering
- Anti-patterns: distributed monolith, nano-services, shared databases
- **Connection to your experience:** Decomposing a monolith into bounded contexts

---

## Part II — Distributed Systems & Data

### Lesson 04: Distributed Systems & Data Management
`lessons/04-distributed-systems-data.md`
- CAP theorem — practical implications (not just theory)
- Consistency models: strong, eventual, causal
- Distributed consensus: Raft, Paxos (conceptual)
- Replication strategies: leader-follower, multi-leader, leaderless
- Partitioning/sharding strategies
- Distributed caching: Redis, Memcached — patterns and pitfalls
- CQRS implementation patterns
- Event sourcing — when to use and when not to
- Change Data Capture (CDC)
- Polyglot persistence
- **Builds on:** DDIA chapters you've read + extends to what you haven't finished
- **Exercise:** Design a data architecture for a large e-commerce platform

### Lesson 10: Event-Driven Architecture & Messaging
`lessons/10-event-driven-architecture.md`
- Event-driven architecture patterns (event notification, event-carried state transfer, event sourcing)
- Message brokers: Apache Kafka, RabbitMQ, cloud-native options (Azure Service Bus, AWS SQS/SNS)
- Kafka deep dive: partitions, consumer groups, exactly-once semantics
- Event schema evolution and compatibility
- Dead letter queues and poison message handling
- Transactional outbox pattern
- Event choreography vs. command orchestration
- Stream processing: Kafka Streams, Apache Flink concepts
- Idempotency patterns
- **Connection to your experience:** Enterprise event systems → Kafka-based architecture

---

## Part III — Cloud Infrastructure & Operations

### Lesson 05: Containers & Orchestration
`lessons/05-containers-orchestration.md`
- Docker deep dive: images, layers, multi-stage builds, security
- Container networking and storage
- Kubernetes architecture: control plane, nodes, pods
- Kubernetes objects: Deployments, Services, ConfigMaps, Secrets, Ingress
- StatefulSets, DaemonSets, Jobs, CronJobs
- Helm charts and package management
- Kubernetes networking: Services, Ingress controllers, Network Policies
- Persistent storage: PV, PVC, StorageClasses
- Resource management: requests, limits, QoS classes
- Kubernetes on cloud: AKS, EKS, GKE — differences and tradeoffs
- **Exercise:** Containerize a Spring Boot app, deploy to Kubernetes

### Lesson 06: Cloud-Native Design Patterns
`lessons/06-cloud-native-patterns.md`
- Circuit breaker (Resilience4j)
- Retry with exponential backoff and jitter
- Bulkhead pattern
- Sidecar / ambassador / adapter patterns
- Strangler fig pattern (migration from monolith)
- Backend for Frontend (BFF)
- Gateway aggregation / routing / offloading
- Health endpoint monitoring
- Leader election
- Competing consumers
- Priority queue
- Throttling / rate limiting
- Valet key pattern
- **Connection to your experience:** Migrating enterprise monoliths using strangler fig

### Lesson 08: Observability & Reliability Engineering
`lessons/08-observability-reliability.md`
- Three pillars: logs, metrics, traces
- Structured logging best practices
- Distributed tracing: OpenTelemetry, Jaeger, Zipkin
- Metrics: Prometheus, Grafana
- SLIs, SLOs, SLAs — defining and measuring reliability
- Error budgets and risk management
- Chaos engineering: principles and tools (Chaos Monkey, Litmus)
- Incident management and post-mortems
- Runbooks and operational readiness
- Golden signals: latency, traffic, errors, saturation
- **Exercise:** Design an observability strategy for a microservices platform

---

## Part IV — Security, APIs, and Cross-Cutting Concerns

### Lesson 07: API Design & Management
`lessons/07-api-design.md`
- REST API maturity model (Richardson)
- API design best practices: versioning, pagination, error handling
- gRPC: Protocol Buffers, streaming, when to use
- GraphQL: benefits, pitfalls, federation
- API gateway patterns: Kong, AWS API Gateway, Azure API Management
- Rate limiting and throttling
- API security: OAuth 2.0, OpenID Connect, JWT, API keys
- API documentation: OpenAPI/Swagger
- API evolution and backward compatibility
- Consumer-driven contract testing (Pact)
- **Connection to your experience:** Enterprise REST APIs → cloud-native API strategy

### Lesson 09: Security in Cloud-Native Systems
`lessons/09-security.md`
- Zero-trust architecture
- Identity and access management (IAM)
- OAuth 2.0 / OpenID Connect flows — deep dive
- Service-to-service authentication: mTLS, JWT propagation
- Secret management: HashiCorp Vault, cloud-native solutions
- Container security: image scanning, runtime protection, pod security
- Network security: network policies, service mesh mTLS
- Supply chain security: SBOM, signed images
- OWASP top 10 for cloud-native
- Compliance as code
- **Exercise:** Design a security architecture for a multi-tenant SaaS platform

---

## Part V — Delivery & Operations

### Lesson 11: CI/CD & GitOps
`lessons/11-cicd-gitops.md`
- CI/CD pipeline design for microservices
- GitOps principles: ArgoCD, Flux
- Infrastructure as Code: Terraform, Pulumi
- Deployment strategies: blue-green, canary, rolling, A/B testing
- Feature flags and progressive delivery
- Environment management: dev, staging, production parity
- Artifact management and container registries
- Pipeline security: SAST, DAST, dependency scanning
- Trunk-based development vs. GitFlow for microservices
- **Exercise:** Design a CI/CD pipeline for a multi-service platform

### Lesson 12: Scalability & Performance
`lessons/12-scalability-performance.md`
- Horizontal vs. vertical scaling
- Auto-scaling: HPA, VPA, KEDA, cluster autoscaling
- Load balancing: L4 vs. L7, algorithms
- CDN and edge computing
- Caching strategies: cache-aside, read-through, write-through, write-behind
- Database scaling: read replicas, sharding, connection pooling
- Asynchronous processing and work queues
- Performance testing: load testing, stress testing, soak testing
- Capacity planning
- Back pressure and flow control
- **Builds on:** Alex Xu system design concepts
- **Exercise:** Design a system to handle 10x traffic spike during Black Friday

---

## Part VI — Architecture Practice

### Lesson 13: Architecture Decision Records & Documentation
`lessons/13-adr-documentation.md`
- Architecture Decision Records (ADRs) — template and examples
- C4 model for architecture documentation
- Architecture diagrams: sequence, component, deployment, context
- Communicating architecture to different audiences
- Architecture fitness functions
- Architecture governance in agile organizations
- Technical debt management and tracking
- Architecture review process
- Documenting non-functional requirements (quality attributes)
- **Exercise:** Write ADRs for a microservices migration project

### Lesson 14: Cost Optimization & FinOps
`lessons/14-cost-optimization-finops.md`
- Cloud cost models: compute, storage, networking, egress
- Right-sizing workloads
- Spot/preemptible instances
- Reserved capacity planning
- Serverless economics
- Cost allocation and tagging strategies
- FinOps framework and practices
- Multi-cloud and cloud-agnostic considerations
- TCO analysis: cloud vs. on-premises
- **Exercise:** Optimize architecture costs for a given workload profile

---

## Recommended Reading Path

### 📚 Books You Should Read Next

#### Must-Read (Priority Order)

1. **"Fundamentals of Software Architecture"** — Mark Richards & Neal Ford
   - *Why:* The best modern overview of architecture styles, patterns, and the architect role. Directly relevant to everything in this curriculum.
   - *When:* Read alongside lessons 01-02

2. **"Building Microservices" (2nd Edition)** — Sam Newman
   - *Why:* The definitive guide to microservices. Covers decomposition, communication, deployment, and organizational aspects.
   - *When:* Read alongside lessons 03-06

3. **Finish "Designing Data-Intensive Applications"** — Martin Kleppmann
   - *Why:* You've read half — the second half covers distributed systems consensus, batch/stream processing, and the future of data systems. Essential for lesson 04 and 10.
   - *When:* Read alongside lesson 04

4. **"Cloud Native Patterns"** — Cornelia Davis
   - *Why:* Practical patterns for building cloud-native applications. Bridges theory and implementation.
   - *When:* Read alongside lesson 06

5. **"Software Architecture: The Hard Parts"** — Neal Ford, Mark Richards, Pramod Sadalage, Zhamak Dehghani
   - *Why:* Tackles the difficult tradeoffs in distributed architectures. Covers data decomposition, service granularity, contracts.
   - *When:* Read after lessons 03-04

#### Highly Recommended

6. **"Domain-Driven Design Distilled"** — Vaughn Vernon
   - *Why:* Concise intro to DDD — critical for microservices decomposition. If you want more depth, follow with the original Evans book.
   - *When:* Read alongside lesson 03

7. **"Release It!" (2nd Edition)** — Michael Nygard
   - *Why:* Stability patterns and anti-patterns for production systems. Circuit breakers, bulkheads, timeouts — all the patterns in lesson 06.
   - *When:* Read alongside lessons 06, 08

8. **"Kubernetes in Action" (2nd Edition)** — Marko Lukša
   - *Why:* Comprehensive Kubernetes guide. Better than most online tutorials for deep understanding.
   - *When:* Read alongside lesson 05

9. **"Team Topologies"** — Matthew Skelton & Manuel Pais
   - *Why:* How to organize teams for fast flow. Conway's Law applied. Architecture and team structure are inseparable.
   - *When:* Read after lesson 03

10. **"The Phoenix Project" / "The Unicorn Project"** — Gene Kim
    - *Why:* Novel format — makes DevOps and flow principles tangible. Good mental model for CI/CD and organizational change.
    - *When:* Read alongside lesson 11

#### Deep Dives (When You Want More)

11. **"Designing Distributed Systems"** — Brendan Burns
    - *Why:* Patterns for distributed systems by Kubernetes co-creator. Sidecar, ambassador, adapter patterns.

12. **"Production-Ready Microservices"** — Susan Fowler
    - *Why:* Operational excellence checklist for microservices.

13. **"Monolith to Microservices"** — Sam Newman
    - *Why:* Practical migration patterns. Strangler fig, branch by abstraction.

14. **"Implementing Domain-Driven Design"** — Vaughn Vernon
    - *Why:* Deep DDD implementation. For when "DDD Distilled" isn't enough.

15. **"System Design Interview Vol 2"** — Alex Xu
    - *Why:* You've read Vol 1. Vol 2 covers more advanced topics: proximity service, nearby friends, stock exchange.

### 🌐 Online Resources

| Resource | URL | Best For |
|----------|-----|----------|
| Microsoft Azure Architecture Center | https://learn.microsoft.com/en-us/azure/architecture/ | Architecture styles, patterns, best practices |
| CNCF Cloud Native Interactive Landscape | https://landscape.cncf.io/ | Understanding the ecosystem |
| Martin Fowler's Blog | https://martinfowler.com/architecture/ | Timeless architecture articles |
| 12factor.net | https://12factor.net/ | The 12-Factor App methodology |
| microservices.io | https://microservices.io/ | Microservices patterns catalog |
| The Architect Elevator | https://architectelevator.com/ | Gregor Hohpe's architecture insights |
| InfoQ Architecture Trends | https://www.infoq.com/architecture-design/ | Current architecture trends |
| ByteByteGo Newsletter | https://blog.bytebytego.com/ | Alex Xu's continued system design content |

### 🎓 Certifications to Consider

| Certification | Provider | Relevance |
|---------------|----------|-----------|
| AWS Solutions Architect (Associate → Professional) | AWS | Most recognized cloud architecture cert |
| Azure Solutions Architect Expert (AZ-305) | Microsoft | Good match given your interest in Azure architecture |
| CKA (Certified Kubernetes Administrator) | CNCF | Validates Kubernetes operational knowledge |
| CKAD (Certified Kubernetes Application Developer) | CNCF | Validates Kubernetes application development |
| iSAQB CPSA-F / CPSA-A | iSAQB | International software architecture certification |

---

## Learning Path Schedule (Suggested)

| Week | Lesson | Parallel Reading |
|------|--------|-----------------|
| 1-2 | 01: Cloud-Native Fundamentals | Fundamentals of Software Architecture (Part I) |
| 3-4 | 02: Architecture Styles | Fundamentals of Software Architecture (Part II) |
| 5-6 | 03: Microservices Deep Dive | Building Microservices (Ch 1-6) |
| 7-8 | 04: Distributed Systems & Data | Finish DDIA |
| 9-10 | 05: Containers & Orchestration | Kubernetes in Action (Part I) |
| 11-12 | 06: Cloud-Native Patterns | Cloud Native Patterns / Release It! |
| 13-14 | 07: API Design | Building Microservices (Ch 7-10) |
| 15-16 | 08: Observability & Reliability | Release It! (Part II) |
| 17-18 | 09: Security | — |
| 19-20 | 10: Event-Driven Architecture | DDIA (Stream processing chapters) |
| 21-22 | 11: CI/CD & GitOps | The Phoenix Project |
| 23-24 | 12: Scalability & Performance | System Design Interview Vol 2 |
| 25-26 | 13: ADRs & Documentation | Software Architecture: The Hard Parts |
| 27-28 | 14: Cost Optimization | Team Topologies |

---

*Created: March 2026*  
*Last updated: March 2026*