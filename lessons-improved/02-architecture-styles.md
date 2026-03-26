# Lesson 02: Architecture Styles & Patterns

> **Prerequisites:** Lesson 01 (Cloud-Native Fundamentals)  
> **Time estimate:** 4-6 hours  
> **Parallel reading:** "Fundamentals of Software Architecture" — Richards & Ford (Part II)

---

## Table of Contents

1. [Architecture Characteristics & Tradeoffs](#1-architecture-characteristics--tradeoffs)
2. [N-Tier / Layered Architecture](#2-n-tier--layered-architecture)
3. [Microservices Architecture](#3-microservices-architecture)
4. [Event-Driven Architecture](#4-event-driven-architecture)
5. [CQRS and Event Sourcing](#5-cqrs-and-event-sourcing)
6. [Other Architecture Styles](#6-other-architecture-styles)
7. [Modular Monolith](#7-modular-monolith)
8. [Architecture Style Decision Framework](#8-architecture-style-decision-framework)
9. [Exercises](#9-exercises)
10. [Self-Check Questions](#10-self-check-questions)
11. [Answers](#11-answers)

---

## 1. Architecture Characteristics & Tradeoffs

### The First Law of Software Architecture

> *"Everything in software architecture is a tradeoff."*  
> — Mark Richards & Neal Ford, "Fundamentals of Software Architecture"

> *"If an architect thinks they have discovered something that isn't a tradeoff, more likely they just haven't identified the tradeoff yet."*  
> — Corollary

### Key Architecture Characteristics (Quality Attributes)

Architecture characteristics (also called "-ilities") define how a system operates, not what it does.

| Characteristic | Definition | Measured By |
|---------------|-----------|-------------|
| **Scalability** | Ability to handle increased load | Requests/sec under load, response time degradation |
| **Availability** | System uptime and accessibility | Uptime % (99.9% = 8.76 hrs downtime/year) |
| **Performance** | Speed of individual operations | Latency (p50, p95, p99), throughput |
| **Reliability** | Consistency of correct behavior | Mean Time Between Failures (MTBF) |
| **Fault Tolerance** | Ability to operate despite failures | Recovery time, degraded mode capabilities |
| **Elasticity** | Auto-scaling to match demand | Scale-up/down time, cost efficiency |
| **Deployability** | Ease of deploying changes | Deployment frequency, lead time, rollback time |
| **Testability** | Ease of testing | Test coverage, test execution time |
| **Modularity** | Degree of component independence | Coupling metrics, change impact radius |
| **Evolvability** | Ease of adding new features | Time to implement new feature, breaking changes |
| **Security** | Protection from threats | Vulnerability count, compliance score |
| **Cost** | Total cost of ownership | Infrastructure + development + operations |

### The Tradeoff Triangle

You cannot maximize all characteristics. Every architecture involves deliberate tradeoffs.

```
                    SCALABILITY
                       /\
                      /  \
                     /    \
                    /      \
                   /  Pick  \
                  /   2-3    \
                 /   primary  \
                /              \
               /________________\
     SIMPLICITY                 PERFORMANCE
     (Maintainability)          (Low Latency)

Example tradeoffs:
• Microservices → ↑ Scalability, ↑ Deployability, ↓ Simplicity
• Monolith     → ↑ Simplicity, ↑ Performance (no network hops), ↓ Scalability
• Event-Driven → ↑ Scalability, ↑ Loose coupling, ↓ Simplicity, ↓ Debuggability
```

---

## 2. N-Tier / Layered Architecture

The most common enterprise architecture style — and likely what you work with today.

### Structure

```
┌─────────────────────────────────────────────┐
│           PRESENTATION LAYER                 │
│  (Controllers, REST endpoints, Views)        │
│  Spring MVC: @RestController, @Controller    │
├─────────────────────────────────────────────┤
│           BUSINESS LOGIC LAYER               │
│  (Services, Domain logic, Validation)        │
│  Spring: @Service, Domain objects             │
├─────────────────────────────────────────────┤
│           PERSISTENCE LAYER                  │
│  (Repositories, Data access, ORM)            │
│  Spring Data JPA: @Repository, JPA Entities  │
├─────────────────────────────────────────────┤
│           DATABASE LAYER                     │
│  (PostgreSQL, MySQL, Oracle)                 │
└─────────────────────────────────────────────┘

Rules:
✅ Each layer only calls the layer directly below it
✅ No skipping layers (controller → repository is forbidden)
❌ Leads to the "architecture sinkhole anti-pattern" — requests pass
   through layers with no logic, just pass-through delegation
```

### Spring Boot Layered Project Structure

```
src/main/java/com/example/orderservice/
├── controller/           # Presentation layer
│   ├── OrderController.java
│   └── dto/
│       ├── CreateOrderRequest.java
│       └── OrderResponse.java
├── service/              # Business logic layer
│   ├── OrderService.java
│   └── OrderValidator.java
├── repository/           # Persistence layer
│   ├── OrderRepository.java
│   └── entity/
│       └── OrderEntity.java
└── config/               # Cross-cutting concerns
    └── SecurityConfig.java
```

### Characteristics

| Characteristic | Rating | Notes |
|---------------|--------|-------|
| Simplicity | ★★★★★ | Easy to understand, well-known by Java developers |
| Deployability | ★★☆☆☆ | Single deployable unit — deploy everything for any change |
| Scalability | ★★☆☆☆ | Only vertical scaling; all layers scale together |
| Testability | ★★★☆☆ | Unit tests easy, integration tests harder |
| Performance | ★★★★☆ | No network overhead between layers (in-process calls) |
| Evolvability | ★★☆☆☆ | Changes often cascade across layers |

### When to Use

✅ Small teams (2-5 developers), simple domain, CRUD-heavy applications, prototypes, MVPs  
❌ Large teams needing independent deployment, high-scale systems, complex domains with many bounded contexts

---

## 3. Microservices Architecture

### Key Characteristics

```
┌─────────────────────────────────────────────────────────┐
│                    API GATEWAY                            │
│  (Routing, auth, rate limiting, SSL termination)         │
└──────┬──────────┬──────────┬──────────┬─────────────────┘
       │          │          │          │
  ┌────┴───┐ ┌───┴────┐ ┌──┴─────┐ ┌─┴──────┐
  │ Order  │ │Payment │ │Inventory│ │Shipping│  ← Independent services
  │Service │ │Service │ │Service  │ │Service │     (own codebase,
  └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘      own team,
      │          │          │          │             own release)
  ┌───┴────┐ ┌──┴─────┐ ┌──┴─────┐ ┌──┴─────┐
  │Order DB│ │Pay DB  │ │Inv DB  │ │Ship DB │  ← Each service
  │(Postgres)│(Postgres)│(MongoDB)│(Postgres)│     owns its data
  └────────┘ └────────┘ └────────┘ └────────┘
```

### The Microservices Premium

Microservices add operational complexity. You pay a "premium" that only makes sense at sufficient scale:

```
COMPLEXITY vs. PRODUCTIVITY

Productivity │
             │     Monolith
             │    ╱
             │   ╱  ← Monolith is simpler and faster for small teams/domains
             │  ╱
             │ ╱
             │╱─────────────── Crossover point
             │╲                (team size ~20-30, or domain complexity threshold)
             │ ╲
             │  ╲  ← Microservices become more productive past this point
             │   ╲
             │    Microservices
             └──────────────────────── Team Size / Domain Complexity
```

**What you need BEFORE microservices:**
1. CI/CD pipeline automation
2. Container orchestration (Kubernetes)
3. Monitoring and distributed tracing
4. Automated testing strategy
5. Team structure aligned with service boundaries (Conway's Law)

### Characteristics

| Characteristic | Rating | Notes |
|---------------|--------|-------|
| Scalability | ★★★★★ | Each service scales independently |
| Deployability | ★★★★★ | Independent deployment per service |
| Evolvability | ★★★★☆ | New features in one service, no impact on others |
| Fault Tolerance | ★★★★☆ | One service failure doesn't bring down the whole system |
| Simplicity | ★★☆☆☆ | Distributed systems complexity, network issues, eventual consistency |
| Testability | ★★★☆☆ | Unit tests easy, end-to-end tests hard |
| Performance | ★★★☆☆ | Network latency between services |

---

## 4. Event-Driven Architecture

### Pub/Sub vs. Event Streaming

```
PUBLISH/SUBSCRIBE (Fan-out)                 EVENT STREAMING (Kafka)
┌──────────┐                                ┌──────────┐
│ Publisher │                                │ Producer │
└────┬─────┘                                └────┬─────┘
     │ Event                                      │ Event
     ▼                                            ▼
┌─────────────┐                             ┌──────────────┐
│  Topic /    │                             │ Kafka Topic  │
│  Exchange   │                             │ (persistent  │
│ (ephemeral) │                             │  log)        │
└──┬───┬───┬──┘                             └──┬───┬───┬──┘
   │   │   │                                   │   │   │
   ▼   ▼   ▼                                   ▼   ▼   ▼
┌──┐ ┌──┐ ┌──┐                             ┌──┐ ┌──┐ ┌──┐
│S1│ │S2│ │S3│  ← Each gets a copy         │CG1│ │CG2│ │CG3│ ← Consumer groups
└──┘ └──┘ └──┘    Message deleted           └──┘ └──┘ └──┘    Events retained
                  after delivery                               (replay possible)

Use cases:                                  Use cases:
• Notifications                             • Event sourcing
• Webhooks                                  • Stream processing
• Simple decoupling                         • Audit log / replay
```

### Three Event Patterns

| Pattern | Description | Example | Data in Event |
|---------|-------------|---------|---------------|
| **Event Notification** | Minimal event signals something happened | `OrderCreated { orderId: 123 }` | Just IDs — consumer calls back for details |
| **Event-Carried State Transfer** | Event contains all data consumer needs | `OrderCreated { orderId: 123, items: [...], total: 99.99, address: {...} }` | Full state — no callbacks needed |
| **Event Sourcing** | Store state as sequence of events | `ItemAdded`, `ItemRemoved`, `OrderPlaced` → replay to get current state | All events are the source of truth |

### Choreography vs. Orchestration

```
CHOREOGRAPHY (event-driven, decentralized)
──────────────────────────────────────────
Order      →  OrderPlaced  →  Payment     →  PaymentCompleted  →  Inventory
Service       (event)         Service        (event)              Service
                                                                      │
              InventoryReserved  ←──────────────────────────────────────┘
              (event)

✅ Loose coupling — services don't know about each other
✅ Easy to add new consumers
❌ Hard to track the overall process flow
❌ Error handling is complex (distributed saga)


ORCHESTRATION (central coordinator)
──────────────────────────────────
                ┌───────────────┐
                │  Order Saga   │  ← Central orchestrator
                │  Orchestrator │     knows the full workflow
                └───┬───┬───┬──┘
                    │   │   │
         ┌──────────┘   │   └──────────┐
         ▼              ▼              ▼
    ┌─────────┐   ┌─────────┐   ┌──────────┐
    │ Payment │   │Inventory│   │ Shipping │
    │ Service │   │ Service │   │ Service  │
    └─────────┘   └─────────┘   └──────────┘

✅ Clear process visibility — orchestrator tracks state
✅ Easier error handling and compensation
❌ Orchestrator is a single point of coupling
❌ Can become a "god service" if not carefully scoped
```

### Characteristics

| Characteristic | Rating | Notes |
|---------------|--------|-------|
| Scalability | ★★★★★ | Async processing, independent consumers |
| Loose Coupling | ★★★★★ | Producers don't know consumers |
| Fault Tolerance | ★★★★☆ | Events can be retried, replayed |
| Simplicity | ★★☆☆☆ | Eventual consistency, event ordering, debugging |
| Performance | ★★★★☆ | Async = non-blocking, but adds latency for "completed" state |
| Testability | ★★☆☆☆ | End-to-end flows are hard to test |

---

## 5. CQRS and Event Sourcing

### CQRS — Command Query Responsibility Segregation

Separate read models from write models.

```
                    ┌──────────────┐
                    │   API Layer  │
                    └──────┬───────┘
                           │
              ┌────────────┴────────────┐
              │                         │
      ┌───────┴────────┐      ┌────────┴───────┐
      │   COMMAND Side │      │   QUERY Side   │
      │   (Write)      │      │   (Read)       │
      ├────────────────┤      ├────────────────┤
      │ Domain Model   │      │ Denormalized   │
      │ (rich, complex)│      │ Read Models    │
      │ Business rules │      │ (optimized for │
      │ Validation     │      │  specific      │
      ├────────────────┤      │  queries)      │
      │ Normalized DB  │      ├────────────────┤
      │ (PostgreSQL)   │      │ Read DB        │
      └───────┬────────┘      │ (Elasticsearch,│
              │               │  Redis, or     │
              │   Events /    │  materialized  │
              └───CDC─────────│  views)        │
                              └────────────────┘

When to use CQRS:
✅ Read and write patterns are very different (e.g., complex writes, simple reads)
✅ Read-heavy workloads (90% reads) — optimize read model independently
✅ Need different data models for reading and writing
❌ Simple CRUD applications — CQRS adds unnecessary complexity
❌ Data that must be immediately consistent after writes
```

### Event Sourcing

Instead of storing current state, store all events that led to the current state.

```
Traditional CRUD:                    Event Sourcing:
┌──────────────┐                    ┌──────────────────────────────┐
│ orders table │                    │ event_store                  │
├──────────────┤                    ├──────────────────────────────┤
│ id: 123      │                    │ 1: OrderCreated {id:123}     │
│ status: PAID │                    │ 2: ItemAdded {sku:"ABC"}     │
│ total: 99.99 │                    │ 3: ItemAdded {sku:"XYZ"}     │
│ items: [...]│                    │ 4: ItemRemoved {sku:"ABC"}   │
└──────────────┘                    │ 5: OrderSubmitted {total:49} │
                                    │ 6: PaymentReceived {amt:49}  │
Current state only.                 └──────────────────────────────┘
History is lost.                    Full history. Replay to get state.

Benefits:
✅ Complete audit trail — every change is recorded
✅ Temporal queries — "what was the state at 2pm yesterday?"
✅ Debug production issues by replaying events
✅ Build new read models by replaying from the beginning

Challenges:
❌ Schema evolution — events stored forever, schema changes over time
❌ Complexity — need snapshots for performance, event versioning
❌ Eventual consistency — read models are updated asynchronously
❌ Not suitable for all domains — simple CRUD doesn't benefit
```

---

## 6. Other Architecture Styles

### Web-Queue-Worker

```
┌──────────┐     ┌─────────┐     ┌──────────┐
│   Web    │ ──→ │  Queue  │ ──→ │  Worker  │
│ Frontend │     │ (SQS,   │     │ (Process │
│ (API)    │     │  RabbitMQ)│    │  tasks)  │
└──────────┘     └─────────┘     └──────────┘

✅ Simple async processing pattern
✅ Web and worker scale independently
✅ Queue provides buffering and decoupling
Use case: Image processing, report generation, email sending
```

### Lambda / Kappa Architecture (Big Data)

```
LAMBDA ARCHITECTURE:
───────────────────
                ┌─── Batch Layer (MapReduce, Spark) ──→ Batch Views ─┐
Raw Data ───────┤                                                     ├──→ Query
                └─── Speed Layer (Storm, Flink)    ──→ Real-time Views┘

KAPPA ARCHITECTURE (simplified):
─────────────────────────────────
Raw Data ──→ Stream Processing (Kafka Streams, Flink) ──→ Views

Kappa removes batch layer — everything is a stream.
Simpler to maintain but requires mature stream processing infrastructure.
```

### Space-Based Architecture

```
┌──────────────────────────────────────────────────┐
│            Processing Grid (In-Memory)            │
├──────────┬──────────┬──────────┬─────────────────┤
│  Unit 1  │  Unit 2  │  Unit 3  │  Unit N         │
│  (App +  │  (App +  │  (App +  │  (App +         │
│   Data)  │   Data)  │   Data)  │   Data)         │
│          │          │          │                  │
│ In-memory│ In-memory│ In-memory│ In-memory        │
│ data grid│ data grid│ data grid│ data grid        │
└────┬─────┴────┬─────┴────┬─────┴─────────────────┘
     │          │          │
     └──────────┼──────────┘
                │ Async replication
         ┌──────┴──────┐
         │  Database   │  ← Persistent store (async write-behind)
         └─────────────┘

✅ Extreme scalability (all data in memory)
✅ No database bottleneck for reads
❌ Complex — data replication, consistency
Use case: Concert ticket sales, real-time bidding, high-frequency trading
```

### Micro-Frontends

```
┌────────────────────────────────────────────────┐
│                 App Shell / Container            │
├──────────┬─────────────┬──────────┬────────────┤
│  Product │   Cart      │  Order   │  Profile   │
│  Team    │   Team      │  Team    │  Team      │
│          │             │          │            │
│ React    │  Vue.js     │  React   │  Angular   │
│ Micro-FE │  Micro-FE   │  Micro-FE│  Micro-FE  │
│          │             │          │            │
│ Product  │  Cart       │  Order   │  Profile   │
│ Service  │  Service    │  Service │  Service   │
│ (API)    │  (API)      │  (API)   │  (API)     │
└──────────┴─────────────┴──────────┴────────────┘

✅ Vertical team ownership (UI + API + DB)
✅ Independent deployment of frontend features
✅ Teams can choose different frontend frameworks
❌ Complexity in integration, shared state, consistent UX
❌ Bundle size overhead from multiple frameworks
```

---

## 7. Modular Monolith

The pragmatic middle ground — especially relevant as a first step before microservices.

### Structure

```
┌─────────────────────────────────────────────┐
│             Single Deployable Unit           │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Order   │  │ Payment  │  │ Inventory│  │
│  │  Module  │  │  Module  │  │  Module  │  │
│  │          │  │          │  │          │  │
│  │ Public   │  │ Public   │  │ Public   │  │
│  │ API      │→ │ API      │→ │ API      │  │
│  │(interface)│  │(interface)│  │(interface)│  │
│  │          │  │          │  │          │  │
│  │ Internal │  │ Internal │  │ Internal │  │
│  │ impl     │  │ impl     │  │ impl     │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
│       │             │             │         │
│  ┌────┴─────┐  ┌────┴─────┐  ┌───┴──────┐  │
│  │Order     │  │Payment   │  │Inventory │  │
│  │Schema    │  │Schema    │  │Schema    │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                              │
│           Shared Database Server             │
│      (but separate schemas per module)       │
└─────────────────────────────────────────────┘

Rules:
✅ Modules communicate ONLY through public APIs (interfaces)
✅ No direct database access across module boundaries
✅ Each module has its own database schema
❌ No cross-module JOINs, no shared tables
```

### Enforcing Module Boundaries with ArchUnit

```java
@AnalyzeClasses(packages = "com.example")
public class ModuleBoundaryTest {

    @ArchTest
    static final ArchRule orderModuleDoesNotDependOnPaymentInternal =
        noClasses()
            .that().resideInAPackage("..order..")
            .should().dependOnClassesThat()
            .resideInAPackage("..payment.internal..")
            .because("Order module must use Payment's public API, not internal classes");

    @ArchTest
    static final ArchRule modulesOnlyAccessOwnRepositories =
        noClasses()
            .that().resideInAPackage("..order..")
            .should().dependOnClassesThat()
            .resideInAPackage("..payment.repository..")
            .because("Modules must not access each other's repositories directly");

    @ArchTest
    static final ArchRule noCircularDependencies =
        slices().matching("com.example.(*)..")
            .should().beFreeOfCycles();
}
```

### When to Choose Modular Monolith

| Scenario | Modular Monolith | Microservices |
|----------|-----------------|---------------|
| Small team (< 10 developers) | ✅ Preferred | ❌ Overkill |
| Domain boundaries unclear | ✅ Easier to refactor | ❌ Wrong boundaries are expensive |
| Startup / MVP | ✅ Ship faster | ❌ Premature optimization |
| Need independent scaling | ❌ All modules scale together | ✅ Per-service scaling |
| Need independent deployment | ❌ Single deployment unit | ✅ Per-service deployment |
| Strong cross-module queries | ✅ In-process joins still possible | ❌ Distributed queries are hard |

---

## 8. Architecture Style Decision Framework

### Comparison Matrix

| Characteristic | Layered | Microservices | Event-Driven | Modular Monolith | Space-Based |
|---------------|---------|--------------|-------------|-----------------|------------|
| **Deployability** | ★★ | ★★★★★ | ★★★★ | ★★★ | ★★★ |
| **Scalability** | ★★ | ★★★★★ | ★★★★★ | ★★★ | ★★★★★ |
| **Simplicity** | ★★★★★ | ★★ | ★★ | ★★★★ | ★★ |
| **Performance** | ★★★★ | ★★★ | ★★★★ | ★★★★ | ★★★★★ |
| **Testability** | ★★★ | ★★★ | ★★ | ★★★★ | ★★ |
| **Evolvability** | ★★ | ★★★★ | ★★★★★ | ★★★ | ★★★ |
| **Fault Tolerance** | ★ | ★★★★ | ★★★★★ | ★★ | ★★★★ |
| **Cost** | ★★★★★ | ★★ | ★★★ | ★★★★ | ★★ |

### Decision Flowchart

```
START: What are your primary architecture drivers?

Q1: Team size?
├── < 10 developers → Consider Modular Monolith or Layered
└── > 10 developers → Continue to Q2

Q2: Do you need independent deployment of components?
├── No → Modular Monolith
└── Yes → Continue to Q3

Q3: Is the system event-driven by nature?
├── Yes → Event-Driven Architecture (+ Microservices for services)
└── No → Continue to Q4

Q4: Do different parts need different scaling strategies?
├── Yes → Microservices
└── No → Continue to Q5

Q5: Is the domain well-understood with clear boundaries?
├── Yes → Microservices
└── No → Modular Monolith (refine boundaries first, extract later)
```

---

## 9. Exercises

### Exercise 1: Architecture Style Mapping

For each system below, identify the most appropriate architecture style and justify your choice:

1. Internal HR portal for a 200-person company (5 developers)
2. Real-time stock trading platform (millisecond latency requirements)
3. E-commerce platform for a fast-growing startup (currently 3 developers, planning to grow to 30)
4. IoT sensor data processing (1M events/second from 100K devices)
5. Government tax filing system (once-a-year peak, strict compliance requirements)

For each: Name the style, list the top 3 architecture characteristics driving the choice, and identify the key tradeoff you're making.

---

### Exercise 2: Architecture Tradeoff Analysis

Your company has a monolithic e-commerce application (Spring Boot, 300K LOC, PostgreSQL). Business wants to:
- Deploy features to the product catalog independently from order processing
- Scale the checkout flow independently during sales events
- Allow the mobile team to work independently from the web team

Design a migration proposal:
1. Which architecture style do you recommend as the target?
2. What is your migration strategy (big bang vs. incremental)?
3. What are the first 3 services you would extract and why?
4. What infrastructure prerequisites do you need before starting?
5. What are the top 3 risks and mitigations?

---

### Exercise 3: CQRS Decision

Your team is building an order management system with these requirements:
- Write: Complex business rules for order creation, modification, cancellation
- Read: Dashboard showing order statistics, searchable order history, real-time order tracking
- Read:write ratio is approximately 10:1

Compare two approaches:
1. **Standard approach:** Single model for reads and writes with PostgreSQL
2. **CQRS approach:** Write model in PostgreSQL, read model in Elasticsearch

For each approach, document:
- Architecture diagram
- Pros and cons
- Complexity cost
- When you would choose this approach

---

### Exercise 4: Event-Driven Design

Design an event-driven order processing system with these services: Order, Payment, Inventory, Shipping, Notification.

1. Define 8-10 events (name + key fields)
2. Draw the event flow for a successful order
3. Draw the event flow for a payment failure (what compensating actions occur?)
4. Decide: choreography or orchestration? Justify your choice.
5. What happens if the Notification service is down when OrderShipped fires?

---

## 10. Self-Check Questions

1. What is the "First Law of Software Architecture" according to Richards & Ford?

2. Name 5 architecture characteristics and explain why you can't maximize all of them simultaneously.

3. What is the "microservices premium"? When does it pay off?

4. Explain the difference between choreography and orchestration in event-driven architecture. When would you choose each?

5. What is CQRS? When is it worth the added complexity?

6. What is a modular monolith? Why might it be a better starting point than microservices?

7. What tools or techniques can you use to enforce module boundaries in a monolith?

8. Your startup CTO says "We should use microservices from day one because we plan to scale." What is your response?

---

## 11. Answers

### Answer 1
*"Everything in software architecture is a tradeoff."* Every architectural decision involves giving up something in favor of something else. There are no free lunches — maximizing scalability often reduces simplicity; maximizing performance often reduces maintainability.

### Answer 2
Five architecture characteristics: **Scalability, Performance, Simplicity, Deployability, Fault Tolerance**. You can't maximize all because they create tensions: microservices maximize scalability and deployability but sacrifice simplicity. A monolith maximizes simplicity and performance (in-process calls) but sacrifices independent scalability and deployability. Event-driven maximizes loose coupling and fault tolerance but makes debugging and testing harder.

### Answer 3
The microservices premium is the additional operational complexity you accept when choosing microservices: you need CI/CD pipelines per service, container orchestration, distributed tracing, service discovery, and teams organized by service. It pays off when: (a) you have enough team size (20+ developers) that independent deployment matters, (b) you need to scale parts of the system independently, (c) the domain is well-understood so you can draw correct service boundaries.

### Answer 4
**Choreography:** Services react to events independently, no central coordinator. Each service publishes events and subscribes to events from others. Choose when: services are truly independent, adding new consumers is frequent, you want maximum decoupling. **Orchestration:** A central coordinator directs the flow, telling each service what to do and handling responses. Choose when: the business process has a clear sequence with complex error handling, you need visibility into the overall process state, compensating transactions are required. Most real-world systems use a combination.

### Answer 5
CQRS separates the write model (optimized for business rules and data integrity) from the read model (optimized for queries and display). Worth the complexity when: read and write patterns differ significantly (e.g., complex writes but simple reads), the system is read-heavy (10:1 or higher), different data representations are needed for reading vs. writing (e.g., normalized writes, denormalized reads for search). Not worth it for simple CRUD applications.

### Answer 6
A modular monolith is a single deployable unit with well-defined internal module boundaries. Each module has a public API (interface), own internal implementation, and (ideally) own database schema. It's better as a starting point because: (a) it's simpler to deploy and operate, (b) in-process communication is faster and simpler, (c) module boundaries can be refined before extracting to microservices, (d) wrong boundaries in a monolith are cheaper to fix than wrong service boundaries in microservices.

### Answer 7
**ArchUnit** — Java library for architecture tests that run in the CI pipeline. Can enforce: no cyclic dependencies between modules, modules only access each other's public APIs, no cross-module repository access. **Java module system** (JPMS since Java 9) — compile-time enforcement of module boundaries with `module-info.java`. **Build tool separation** — separate Maven/Gradle modules with explicit dependency declarations.

### Answer 8
"Premature microservices is premature optimization at the architecture level. You don't yet know your domain boundaries well enough to draw correct service lines — wrong boundaries in microservices are expensive to fix. Start with a well-structured modular monolith: enforce module boundaries with ArchUnit, separate database schemas per module, and communicate through defined interfaces. When you have 10+ developers and clear bounded contexts, extract services one at a time using the strangler fig pattern. This gives you the option to go microservices when the time is right, without paying the operational complexity tax upfront."

---

## Key Takeaways

1. **Architecture is about tradeoffs**, not best practices. Document the tradeoffs you're making and why.
2. **The modular monolith is underrated.** For most teams, it's the best starting point — simpler to operate than microservices, with the ability to extract services later.
3. **Microservices require operational maturity.** Don't adopt microservices until you have CI/CD, monitoring, and team structures to support them.
4. **Event-driven architecture is powerful but complex.** Eventual consistency, event ordering, and debugging distributed flows require investment.
5. **CQRS adds value when reads and writes differ significantly.** For simple CRUD, it's overkill.

---

## References

- "Fundamentals of Software Architecture" — Mark Richards & Neal Ford
- "Building Microservices" (2nd Ed.) — Sam Newman
- [Microsoft Architecture Styles Guide](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/)
- [Martin Fowler — CQRS](https://martinfowler.com/bliki/CQRS.html)
- [C4 Model](https://c4model.com/) — Architecture diagramming
- [ArchUnit](https://www.archunit.org/) — Architecture tests for Java

---

*Previous lesson: [01 — Cloud-Native Fundamentals](01-cloud-native-fundamentals.md)*  
*Next lesson: [03 — Microservices Deep Dive](03-microservices-deep-dive.md)*