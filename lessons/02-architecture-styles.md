# Lesson 02: Architecture Styles & Patterns

> **Prerequisites:** Lesson 01 — Cloud-Native Fundamentals  
> **Time estimate:** 6-8 hours  
> **Parallel reading:** "Fundamentals of Software Architecture" — Richards & Ford (Part II)  
> **Reference:** [Microsoft Architecture Styles Guide](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/)

---

## Table of Contents

1. [How to Think About Architecture Styles](#1-how-to-think-about-architecture-styles)
2. [N-Tier / Layered Architecture](#2-n-tier--layered-architecture)
3. [Microservices Architecture](#3-microservices-architecture)
4. [Event-Driven Architecture](#4-event-driven-architecture)
5. [CQRS & Event Sourcing](#5-cqrs--event-sourcing)
6. [Web-Queue-Worker](#6-web-queue-worker)
7. [Big Data Architectures](#7-big-data-architectures)
8. [Choreography vs. Orchestration](#8-choreography-vs-orchestration)
9. [Space-Based Architecture](#9-space-based-architecture)
10. [Micro-Frontends](#10-micro-frontends)
11. [Choosing the Right Style](#11-choosing-the-right-style)
12. [Exercises](#12-exercises)
13. [Self-Check Questions](#13-self-check-questions)
14. [Answers](#14-answers)

---

## 1. How to Think About Architecture Styles

Architecture styles are **not mutually exclusive**. Most real systems combine multiple styles. Think of them as a **palette of tools** — the architect's job is choosing the right combination.

### Architecture Characteristics (Quality Attributes)

Every architecture style optimizes for different quality attributes:

| Quality Attribute | Description |
|-------------------|-------------|
| **Scalability** | Handle increased load by adding resources |
| **Elasticity** | Dynamically scale up/down based on load |
| **Availability** | Percentage of time the system is operational |
| **Reliability** | Probability of failure-free operation |
| **Performance** | Response time, throughput, latency |
| **Deployability** | Ease and frequency of deployments |
| **Testability** | Ease of testing individual components |
| **Modularity** | Degree of component independence |
| **Fault tolerance** | Continue operating despite component failures |
| **Cost** | Infrastructure and operational cost |
| **Simplicity** | Ease of understanding and maintenance |
| **Evolvability** | Ease of adapting to new requirements |

### The Architecture Tradeoff

> **There is no "best" architecture.** Every choice is a tradeoff.

```
"Everything in software architecture is a tradeoff."
  — First Law of Software Architecture (Richards & Ford)

"If an architect thinks they have discovered something that is NOT 
a tradeoff, more likely they just haven't identified the tradeoff yet."
  — Second Law of Software Architecture
```

### Architecture Styles Overview Map

```
                    MONOLITHIC                    DISTRIBUTED
                    ──────────                    ───────────
                    
Simple ←──────────────────────────────────────────────────→ Complex
Cheap ←───────────────────────────────────────────────────→ Expensive

┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌──────────┐
│  Layered  │  │  Modular  │  │  Service  │  │  Micro-   │  │ Serverless│
│  Monolith │  │  Monolith │  │  Based    │  │  services │  │           │
│           │  │           │  │           │  │           │  │           │
│ Simplest  │  │ Better    │  │ Domain    │  │ Full      │  │ Ultimate  │
│ starting  │  │ structure │  │ services  │  │ decentra- │  │ elasticity│
│ point     │  │ and       │  │ larger    │  │ lization  │  │           │
│           │  │ boundaries│  │ than µsvc │  │           │  │           │
└──────────┘  └──────────┘  └──────────┘  └───────────┘  └──────────┘

Deployability:  Low ─────────────────────────────────────→ High
Scalability:    Low ─────────────────────────────────────→ High
Fault tolerance:Low ─────────────────────────────────────→ High
Simplicity:     High ────────────────────────────────────→ Low
Cost:           Low ─────────────────────────────────────→ High
```

---

## 2. N-Tier / Layered Architecture

### Overview

The most common starting architecture. Organizes code into horizontal layers, each with a specific responsibility.

```
┌─────────────────────────────┐
│      Presentation Layer      │  ← UI, REST Controllers
├─────────────────────────────┤
│       Business Layer         │  ← Services, domain logic
├─────────────────────────────┤
│      Persistence Layer       │  ← Repositories, DAOs
├─────────────────────────────┤
│       Database Layer         │  ← SQL, stored procedures
└─────────────────────────────┘

Rules:
- Each layer only depends on the layer below
- Requests flow top-down
- Open layers can be bypassed; closed layers cannot
```

### Spring Boot Example (Classic 3-Tier)

```
my-app/
├── controller/           ← Presentation layer
│   └── OrderController.java
├── service/              ← Business layer
│   └── OrderService.java
├── repository/           ← Persistence layer
│   └── OrderRepository.java
├── model/                ← Domain objects
│   └── Order.java
└── MyApplication.java
```

### Strengths & Weaknesses

| ✅ Strengths | ❌ Weaknesses |
|-------------|--------------|
| Simple, well-understood | Monolithic deployment |
| Good for small teams | Horizontal layers encourage broad changes |
| Low cost | Difficult to scale independently |
| Easy testing of individual layers | "Architecture sinkhole" anti-pattern |
| Good starting point | Tight coupling between layers grows over time |

### Architecture Sinkhole Anti-Pattern

When requests pass through layers without any logic — just passthrough delegation:

```java
// Controller → just delegates
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderService.getOrder(id);  // No controller logic
}

// Service → just delegates  
public Order getOrder(Long id) {
    return orderRepository.findById(id).orElseThrow();  // No business logic
}
```

**Rule of thumb:** If more than 20% of requests are sinkholes, consider a different architecture.

### Enterprise Monolith Context

A typical enterprise Java application uses layered architecture with:
- Web layer (controllers, REST endpoints, pages)
- Facade layer (facade design pattern, DTOs)
- Service layer (business logic, transaction management)
- DAO/Repository layer (data access, JPA/Hibernate)
- Domain model layer (entities, value objects)

This is a well-structured layered monolith — but tight coupling between modules through the shared data model and database prevents independent scaling and deployment. Breaking out of the layered monolith requires first establishing clear module boundaries (modular monolith) before extracting microservices.

---

## 3. Microservices Architecture

### Overview

An architecture style where the application is composed of small, independent services that communicate over the network.

```
┌─────────┐     ┌──────────┐     ┌──────────┐
│  API     │     │  Order   │     │ Inventory │
│  Gateway │────→│  Service │────→│  Service  │
│          │     │          │     │           │
│          │     │ Own DB   │     │  Own DB   │
└─────────┘     └──────────┘     └──────────┘
      │              │                 │
      │         ┌────┴─────┐          │
      │         │ Message  │          │
      │         │  Broker  │──────────┘
      │         └──────────┘
      │              │
      │         ┌────┴─────┐
      └────────→│ Payment  │
                │ Service  │
                │          │
                │ Own DB   │
                └──────────┘
```

### Key Characteristics

1. **Single responsibility** — each service does one thing well
2. **Own data** — each service has its own database
3. **Independent deployment** — deploy one service without touching others
4. **Technology diversity** — each service can use different tech stack
5. **Decentralized governance** — teams own their services end-to-end
6. **Smart endpoints, dumb pipes** — business logic in services, not middleware

### When to Use vs. When to Avoid

| ✅ Use When | ❌ Avoid When |
|------------|--------------|
| Large application with clear domain boundaries | Small, simple application |
| Multiple teams need to work independently | Small team (< 8 people) |
| Different parts need different scaling | Domain boundaries are unclear |
| Need frequent, independent deployments | No DevOps/CI/CD maturity |
| Different components have different tech needs | Tight budget for infrastructure |

### The "Microservices Premium"

Microservices add significant complexity. You pay a "premium" for:
- Network communication overhead
- Distributed data management
- Operational complexity (monitoring, tracing, debugging)
- Testing complexity
- Deployment infrastructure (Kubernetes, CI/CD)

> **"Don't even consider microservices unless you have a system that's too complex to manage as a monolith."** — Martin Fowler

### Microsoft Reference Architecture

From the [Azure Architecture Guide](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/microservices):

```
┌──────────────────────────────────────────────┐
│                  Clients                      │
└──────────────┬───────────────────────────────┘
               │
┌──────────────┴───────────────────────────────┐
│              API Gateway                      │
│  (routing, auth, rate limiting, SSL)          │
└──────┬───────────┬───────────┬───────────────┘
       │           │           │
┌──────┴──┐  ┌────┴────┐  ┌──┴───────┐
│Service A│  │Service B│  │Service C │
│         │  │         │  │          │
│ ┌─────┐ │  │ ┌─────┐ │  │ ┌──────┐ │
│ │DB A │ │  │ │DB B │ │  │ │DB C  │ │
│ └─────┘ │  │ └─────┘ │  │ └──────┘ │
└─────────┘  └─────────┘  └──────────┘
       │           │           │
┌──────┴───────────┴───────────┴───────────────┐
│           Message Broker / Event Bus          │
└──────────────────────────────────────────────┘
```

*(Covered in much more depth in Lesson 03)*

---

## 4. Event-Driven Architecture

### Overview

An architecture style where components communicate by producing and consuming events. Components are loosely coupled — producers don't know about consumers.

### Two Main Models

#### Model 1: Event Notification (Pub/Sub)

```
┌─────────┐   OrderPlaced    ┌───────────┐
│  Order   │ ───────────────→ │  Email    │
│  Service │                  │  Service  │
└─────────┘        │          └───────────┘
                   │
                   │          ┌───────────┐
                   └────────→ │ Inventory │
                              │  Service  │
                              └───────────┘
                   │
                   │          ┌───────────┐
                   └────────→ │ Analytics │
                              │  Service  │
                              └───────────┘

Events are "fire and forget"
Producers don't know or care who consumes
Easy to add new consumers
```

#### Model 2: Event Streaming

```
┌─────────┐                      ┌───────────┐
│Producer 1│──→                   │Consumer A │
└─────────┘   ┌───────────────┐  └───────────┘
              │  Event Stream  │
┌─────────┐  │ (Kafka Topic)  │  ┌───────────┐
│Producer 2│──→│               │──→│Consumer B │
└─────────┘  │  ┌───┬───┬───┐│  └───────────┘
              │  │ E │ E │ E ││
              │  │ 1 │ 2 │ 3 ││  ┌───────────┐
              │  └───┴───┴───┘│──→│Consumer C │
              └───────────────┘  └───────────┘

Events are stored and can be replayed
Consumers can read at their own pace
Supports event sourcing
```

### Key Differences Between the Two Models

| Aspect | Pub/Sub (Event Notification) | Event Streaming |
|--------|------------------------------|-----------------|
| Storage | Events are transient | Events are persistent (log) |
| Replay | No — once consumed, gone | Yes — consumers can replay |
| Ordering | No guaranteed ordering | Ordering within partition |
| Consumer coupling | Consumer must be available | Consumer can be offline |
| Example tech | RabbitMQ, Azure Service Bus | Apache Kafka, AWS Kinesis |

### Event Types

| Type | Description | Example |
|------|-------------|---------|
| **Event Notification** | Something happened. Minimal data. | `OrderPlaced { orderId: 123 }` |
| **Event-Carried State Transfer** | Something happened, here's all the data you need. | `OrderPlaced { orderId: 123, items: [...], total: 99.99, customer: {...} }` |
| **Event Sourcing** | State changes stored as sequence of events. | `ItemAdded`, `ItemRemoved`, `OrderSubmitted`, `PaymentReceived` |

### Strengths & Weaknesses

| ✅ Strengths | ❌ Weaknesses |
|-------------|--------------|
| Extreme loose coupling | Harder to reason about flow |
| Easy to add new consumers | Eventual consistency (not immediate) |
| Natural fit for async processing | Debugging distributed events is complex |
| Excellent for real-time systems | Event ordering challenges |
| Resilient — components fail independently | Need dead letter queues for failed events |

*(Covered in much more depth in Lesson 10)*

---

## 5. CQRS & Event Sourcing

### CQRS — Command Query Responsibility Segregation

Separate the write model (commands) from the read model (queries).

```
Traditional:
┌─────────────┐       ┌─────────────┐
│   Service    │──────→│  Database   │
│ (Read+Write) │←──────│  (Single)   │
└─────────────┘       └─────────────┘

CQRS:
                       ┌─────────────┐     ┌──────────┐
              Write ──→│ Write Model │────→│ Write DB │
              (Commands)│ (Domain)    │     │(Normalized)│
┌─────────┐           └─────────────┘     └──────┬───┘
│  Client  │                                      │ Sync
└─────────┘                                       │(Event/CDC)
              Read  ──→┌─────────────┐     ┌──────┴───┐
              (Queries)│ Read Model  │←────│ Read DB  │
                       │ (Projections)│     │(Optimized)│
                       └─────────────┘     └──────────┘
```

### Why CQRS?

1. **Read and write models have different needs.** Writes need validation, business rules, normalization. Reads need fast queries, denormalized views, specific shapes.
2. **Scale reads and writes independently.** Most apps are read-heavy (10:1 or 100:1).
3. **Optimize each side.** Write: relational DB for consistency. Read: document DB, search index, or cache for speed.

### When to Use CQRS

| ✅ Good Fit | ❌ Poor Fit |
|------------|-----------|
| Read/write ratio is very skewed | Simple CRUD applications |
| Complex domain with rich business rules | Small-scale applications |
| Need different read optimizations | Team unfamiliar with eventual consistency |
| Event sourcing is involved | Strong consistency required everywhere |

### Event Sourcing

Instead of storing current state, store the **sequence of events** that led to the current state.

```
Traditional (State-Based):
┌──────────────────────────────────┐
│ Order #123                        │
│ Status: Shipped                   │
│ Total: $99.99                     │
│ Items: [Widget x2, Gadget x1]    │
│ Updated: 2024-03-15               │
└──────────────────────────────────┘

Event Sourced:
┌──────────────────────────────────┐
│ Event 1: OrderCreated            │
│   { orderId: 123, customer: ... } │
├──────────────────────────────────┤
│ Event 2: ItemAdded               │
│   { item: "Widget", qty: 2 }     │
├──────────────────────────────────┤
│ Event 3: ItemAdded               │
│   { item: "Gadget", qty: 1 }     │
├──────────────────────────────────┤
│ Event 4: OrderSubmitted          │
│   { total: $99.99 }              │
├──────────────────────────────────┤
│ Event 5: PaymentReceived         │
│   { amount: $99.99, method: CC }  │
├──────────────────────────────────┤
│ Event 6: OrderShipped            │
│   { tracking: "1Z999..." }       │
└──────────────────────────────────┘

Current state = replay all events
```

### Benefits of Event Sourcing

1. **Complete audit trail** — every state change is recorded
2. **Time travel** — reconstruct state at any point in time
3. **Debugging** — replay events to reproduce bugs
4. **Event replay** — rebuild read models, create new projections
5. **Domain insight** — events reveal what actually happened in the business

### Risks of Event Sourcing

1. **Complexity** — significant learning curve
2. **Event schema evolution** — changing event structure over time is hard
3. **Eventual consistency** — read models lag behind writes
4. **Storage growth** — event log grows indefinitely (snapshotting helps)
5. **Replay time** — rebuilding state from millions of events is slow (snapshotting helps)

---

## 6. Web-Queue-Worker

### Overview

A simple but effective pattern for separating synchronous web processing from asynchronous background work.

```
┌──────────┐     HTTP     ┌──────────┐
│  Client   │────────────→│   Web    │
└──────────┘              │  Front   │
                          │  End     │
                          └────┬─────┘
                               │
                          ┌────┴─────┐
                          │  Queue   │
                          │ (SQS,    │
                          │  RabbitMQ)│
                          └────┬─────┘
                               │
                          ┌────┴─────┐
                          │  Worker  │
                          │          │
                          │ (Process │
                          │  async   │
                          │  tasks)  │
                          └──────────┘
```

### When to Use

- Applications with time-consuming backend processing
- Email sending, report generation, image processing
- Workloads where user doesn't need immediate result
- When you need to decouple request handling from processing

### Microsoft Reference

From [Azure Architecture Guide — Web-Queue-Worker](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/web-queue-worker):

This is often the simplest cloud architecture to implement. It's a great starting point before considering microservices.

### Example: E-Commerce Order Processing

```
User places order → Web accepts, returns "Order received" immediately
                  → Message queued: { orderId: 123, action: "process" }
                  
Worker picks up message:
  1. Validate inventory
  2. Process payment  
  3. Send confirmation email
  4. Update order status

If worker fails → message returns to queue → retry
```

### Strengths & Weaknesses

| ✅ Strengths | ❌ Weaknesses |
|-------------|--------------|
| Simple to understand and implement | Limited to simple architectures |
| Natural separation of sync/async | Queue becomes single point of failure |
| Workers scale independently | Not suitable for complex workflows |
| Built-in retry via queue | Hard to manage complex dependencies |
| Good stepping stone to microservices | |

---

## 7. Big Data Architectures

### Lambda Architecture

Process data through two parallel paths: batch (accurate, slow) and speed (fast, approximate).

```
              ┌──────────────────────────────────────┐
              │          Data Sources                  │
              └──────────────┬───────────────────────┘
                             │
              ┌──────────────┴───────────────────────┐
              │                                       │
     ┌────────┴────────┐               ┌─────────────┴──┐
     │  Batch Layer     │               │  Speed Layer    │
     │  (MapReduce,     │               │  (Storm, Spark  │
     │   Spark Batch)   │               │   Streaming)    │
     │                  │               │                 │
     │  Processes ALL   │               │  Processes NEW  │
     │  historical data │               │  data in        │
     │  periodically    │               │  real-time      │
     └────────┬────────┘               └────────┬───────┘
              │                                  │
     ┌────────┴────────┐               ┌────────┴───────┐
     │   Batch Views    │               │  Real-time     │
     │   (Accurate)     │               │  Views (Fast)  │
     └────────┬────────┘               └────────┬───────┘
              │                                  │
              └──────────────┬───────────────────┘
                             │
                    ┌────────┴────────┐
                    │  Serving Layer   │
                    │  (Merge batch +  │
                    │   real-time)     │
                    └─────────────────┘
```

### Kappa Architecture

Simplified alternative — use streaming for everything. No separate batch layer.

```
              ┌──────────────────────────┐
              │      Data Sources         │
              └───────────┬──────────────┘
                          │
              ┌───────────┴──────────────┐
              │   Stream Processing       │
              │   (Kafka + Flink/KStreams) │
              │                           │
              │   Single pipeline for     │
              │   both real-time and       │
              │   historical reprocessing  │
              └───────────┬──────────────┘
                          │
              ┌───────────┴──────────────┐
              │      Serving Layer        │
              └──────────────────────────┘

Reprocessing: replay the stream from the beginning
```

### Lambda vs. Kappa

| Aspect | Lambda | Kappa |
|--------|--------|-------|
| Complexity | High — two codebases | Lower — single codebase |
| Accuracy | Batch ensures correctness | Depends on stream quality |
| Latency | Batch: high, Speed: low | Always low |
| Reprocessing | Built-in via batch | Replay stream |
| Best for | Complex analytics requiring both speed and accuracy | When streaming alone is sufficient |

### Your DDIA Connection

Kleppmann covers batch and stream processing in the later chapters of DDIA. If you haven't reached those chapters yet, this lesson provides the architectural context — the book provides the deep implementation details.

---

## 8. Choreography vs. Orchestration

Two fundamental approaches to coordinating work across multiple services.

### Orchestration (Central Coordinator)

```
                    ┌─────────────┐
                    │  Order Saga  │
                    │ Orchestrator │
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   ┌──────────┐    ┌──────────┐    ┌──────────┐
   │ Payment  │    │ Inventory│    │ Shipping │
   │ Service  │    │ Service  │    │ Service  │
   └──────────┘    └──────────┘    └──────────┘

The orchestrator tells each service what to do and when.
Like a conductor leading an orchestra.
```

### Choreography (Decentralized Events)

```
   ┌──────────┐  OrderPlaced  ┌──────────┐  PaymentReceived  ┌──────────┐
   │  Order   │ ────────────→ │ Payment  │ ─────────────────→│ Inventory│
   │ Service  │               │ Service  │                   │ Service  │
   └──────────┘               └──────────┘                   └──────────┘
                                                                   │
                                                          InventoryReserved
                                                                   │
                                                                   ▼
                                                            ┌──────────┐
                                                            │ Shipping │
                                                            │ Service  │
                                                            └──────────┘

Each service reacts to events and publishes its own events.
Like dancers who know their own routines and react to each other.
```

### Comparison

| Aspect | Orchestration | Choreography |
|--------|--------------|--------------|
| Coupling | Services coupled to orchestrator | Services loosely coupled via events |
| Visibility | Easy — look at orchestrator | Hard — flow scattered across services |
| Single point of failure | Orchestrator is SPOF | No SPOF |
| Complexity | Centralized — easier to manage | Distributed — harder to track |
| Adding new steps | Modify orchestrator | Add new event consumer |
| Error handling | Centralized compensation | Each service handles its own |
| Best for | Complex workflows with conditions | Simple flows, high autonomy |

### Saga Pattern (Distributed Transactions)

In microservices, you can't use ACID transactions across services. The Saga pattern manages distributed transactions as a sequence of local transactions with compensating actions.

```
Orchestrated Saga:
  1. Create Order (pending)
  2. → Reserve Inventory     ← if fails: Cancel Order
  3. → Process Payment       ← if fails: Release Inventory, Cancel Order
  4. → Schedule Shipping     ← if fails: Refund Payment, Release Inventory, Cancel Order
  5. → Confirm Order

Each step has a compensating action for rollback.
```

*(Covered in much more depth in Lesson 03)*

---

## 9. Space-Based Architecture

### Overview

Designed for applications with high and unpredictable concurrent user loads. Removes the database as a bottleneck by distributing processing and data across multiple nodes.

```
┌─────────────────────────────────────────────────────┐
│                   Virtualized Middleware              │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ Messaging    │  │ Data Grid    │  │ Processing │ │
│  │ Grid         │  │ (Distributed │  │ Grid       │ │
│  │              │  │  Cache)      │  │            │ │
│  └─────────────┘  └──────────────┘  └────────────┘ │
└────────┬────────────────┬──────────────────┬────────┘
         │                │                  │
    ┌────┴────┐     ┌────┴────┐        ┌───┴─────┐
    │Processing│     │Processing│        │Processing│
    │ Unit 1   │     │ Unit 2   │        │ Unit 3   │
    │          │     │          │        │          │
    │ App +    │     │ App +    │        │ App +    │
    │ In-Memory│     │ In-Memory│        │ In-Memory│
    │ Data Grid│     │ Data Grid│        │ Data Grid│
    └──────────┘     └──────────┘        └─────────┘
                          │
                   ┌──────┴──────┐
                   │ Data Pump   │  (Async writes to DB)
                   │             │
                   └──────┬──────┘
                          │
                   ┌──────┴──────┐
                   │  Database   │  (System of record)
                   └─────────────┘
```

### Key Concepts

- **Processing Units:** Self-contained units with application logic + in-memory data
- **Data Grid:** Distributed cache that keeps data synchronized across units
- **Data Pump:** Asynchronously persists data to the database
- **No database bottleneck:** The database is not in the request path

### When to Use

- Concert ticketing systems
- Auction platforms
- Online trading systems
- Any system with extreme, spiky load

### Strengths & Weaknesses

| ✅ Strengths | ❌ Weaknesses |
|-------------|--------------|
| Extreme scalability | Very complex to implement |
| No database bottleneck | Data consistency challenges |
| Handle spiky loads well | Expensive infrastructure |
| High performance | Not suitable for most applications |

---

## 10. Micro-Frontends

### Overview

Apply microservices principles to the frontend. Each team owns a vertical slice — from UI to database.

```
┌─────────────────────────────────────────────────────┐
│                    Application Shell                  │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │  Product     │  │   Cart       │  │  Account   │ │
│  │  Micro-FE   │  │   Micro-FE   │  │  Micro-FE  │ │
│  │  (React)    │  │   (Vue)      │  │  (Angular) │ │
│  └──────┬──────┘  └──────┬───────┘  └─────┬──────┘ │
└─────────┼────────────────┼─────────────────┼────────┘
          │                │                 │
     ┌────┴────┐     ┌────┴────┐       ┌───┴─────┐
     │ Product │     │  Cart   │       │ Account │
     │ Service │     │ Service │       │ Service │
     │ (Team A)│     │(Team B) │       │(Team C) │
     └─────────┘     └─────────┘       └─────────┘

Each team: owns UI + API + data + deployment
```

### Integration Approaches

| Approach | Description | Pros | Cons |
|----------|-------------|------|------|
| **Build-time** | NPM packages composed at build | Type safety, simple | Coupled deployments |
| **Server-side** | SSI, ESI, or server-composed HTML | Good SEO, fast | Complex server setup |
| **Run-time (iframe)** | Each micro-FE in an iframe | Full isolation | Poor UX, styling challenges |
| **Run-time (JS)** | Module Federation (Webpack 5) | Independent deploy, shared deps | Complex configuration |
| **Web Components** | Custom elements | Framework agnostic | Browser support, style encapsulation |

### When to Use

- Large teams (5+ teams working on same frontend)
- Need independent frontend deployments
- Different parts of the UI have different technology needs
- Legacy migration (wrap old UI in micro-FE)

---

## 11. Choosing the Right Style

### Decision Framework

```
START
  │
  ├─ Is the application simple CRUD with < 5 entities?
  │   └─ YES → Layered Monolith
  │
  ├─ Is there one team (< 8 people)?
  │   └─ YES → Modular Monolith (with clear module boundaries)
  │
  ├─ Do different parts need independent scaling?
  │   └─ YES → Microservices
  │
  ├─ Is there heavy async processing?
  │   └─ YES → Web-Queue-Worker or Event-Driven
  │
  ├─ Are read/write patterns very different?
  │   └─ YES → Consider CQRS
  │
  ├─ Need complete audit trail of changes?
  │   └─ YES → Consider Event Sourcing
  │
  ├─ Extreme spiky loads (ticketing, auctions)?
  │   └─ YES → Space-Based Architecture
  │
  └─ Multiple teams need frontend autonomy?
      └─ YES → Micro-Frontends
```

### Architecture Style Comparison Matrix

| Style | Scalability | Simplicity | Cost | Deployability | Testability |
|-------|-------------|------------|------|---------------|-------------|
| Layered | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐ |
| Modular Monolith | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| Microservices | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Event-Driven | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| Web-Queue-Worker | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| CQRS | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Space-Based | ⭐⭐⭐⭐⭐ | ⭐ | ⭐ | ⭐⭐ | ⭐⭐ |

---

## 12. Exercises

### Exercise 1: Architecture Style Mapping

For each of the following systems, identify the most appropriate architecture style(s) and justify your choice:

1. A corporate expense reporting tool (200 users, simple CRUD)
2. An e-commerce platform handling 100K orders/day with seasonal spikes
3. A real-time stock trading platform
4. A social media analytics dashboard processing millions of events/day
5. A legacy banking system being modernized incrementally
6. An IoT platform collecting sensor data from 100K devices

### Exercise 2: Architecture Tradeoff Analysis

You are the architect for a mid-size e-commerce company. The CTO wants to move from a monolith to microservices. The company has:
- 30 developers in 4 teams
- A Spring Boot monolith with 500K lines of code
- A single PostgreSQL database
- Manual deployments every 2 weeks
- No container orchestration

Write an architecture proposal that:
1. Identifies the top 3 architecture styles to consider
2. Evaluates tradeoffs for each
3. Recommends a migration path (not big-bang rewrite)
4. Identifies prerequisites (what must be in place before migrating)

### Exercise 3: CQRS Decision

Your team is building a product catalog service for an e-commerce platform:
- 50M products
- 10K updates/day
- 10M searches/day (100:1 read/write ratio)
- Search needs: full-text, faceted, geographic
- Writes need: validation, approval workflows

Should you use CQRS? Design both the CQRS and non-CQRS approaches. Compare them on:
- Complexity
- Performance
- Consistency
- Operational cost

### Exercise 4: Event-Driven Design

Design an event-driven architecture for an order processing system with these requirements:
- Order placement
- Payment processing
- Inventory management
- Shipping coordination
- Email notifications
- Analytics/reporting

Document:
1. The events that flow through the system
2. Which services produce and consume each event
3. Whether you'd use choreography or orchestration (and why)
4. How you'd handle failure scenarios (payment fails, inventory insufficient)

---

## 13. Self-Check Questions

1. Name five architecture quality attributes and explain how they might conflict with each other.

2. What is the "architecture sinkhole" anti-pattern in layered architecture? How do you know if your application has it?

3. When would you recommend a modular monolith over microservices? Give three specific criteria.

4. Explain the difference between event notification and event-carried state transfer. When would you choose each?

5. A colleague says: "We should use event sourcing for all our services." Why is this likely a bad idea? When IS event sourcing a good choice?

6. Compare choreography and orchestration. Design a simple order flow using each approach. Which would you prefer for: (a) 3 services, (b) 10 services?

7. What is the relationship between CQRS and Event Sourcing? Can you use one without the other?

8. Your team needs to choose between Lambda and Kappa architecture for a data pipeline. The pipeline must process 1TB/day of clickstream data for both real-time dashboards and monthly reports. Which would you recommend and why?

9. When would you use the Web-Queue-Worker pattern instead of microservices?

10. What does "smart endpoints, dumb pipes" mean in the context of microservices? How does this differ from SOA?

---

## 14. Answers

### Answer 1

Five quality attributes and their conflicts:
1. **Performance** vs. **Scalability** — Optimizing for single-request performance (caching, denormalization) can make horizontal scaling harder
2. **Availability** vs. **Consistency** — CAP theorem: in a partition, you must choose (eventual consistency vs. strong consistency)
3. **Simplicity** vs. **Evolvability** — Simple architectures (monolith) are easy to understand but hard to evolve. Complex architectures (microservices) are evolvable but hard to understand.
4. **Security** vs. **Performance** — Encryption, authentication, authorization all add latency
5. **Cost** vs. **Scalability** — Highly scalable architectures (microservices, space-based) cost more to build and operate

### Answer 2

The architecture sinkhole anti-pattern occurs when requests pass through multiple layers without any meaningful processing — just passthrough calls. 

**How to detect it:** If more than 20% of your requests simply forward data through layers without any transformation, validation, or business logic at each layer, you have the sinkhole problem.

**Example:** `Controller.getUser()` → `Service.getUser()` → `Repository.findById()` — the service layer adds nothing.

**Solutions:**
- Allow some layer bypassing (open layers) for simple CRUD
- Consider a different architecture (CQRS, or moving to vertical slices)
- Don't force every request through every layer if it doesn't add value

### Answer 3

Recommend a modular monolith over microservices when:

1. **Small team size** (< 8-10 developers) — Microservices need team-per-service ownership. A small team managing 15 microservices will spend more time on infrastructure than business logic.

2. **Unclear domain boundaries** — If you don't yet understand where the natural service boundaries are, premature decomposition leads to a "distributed monolith." A modular monolith lets you establish boundaries in code first, then extract services when boundaries are proven.

3. **Low operational maturity** — No CI/CD, no container orchestration, no distributed tracing, no centralized logging. Microservices without this infrastructure is chaos. Build the modular monolith, build the operational foundation, then extract services.

**Additional criteria:** Low deployment frequency needs, tight budget, application with primarily synchronous workflows, strong consistency requirements.

### Answer 4

**Event Notification:** The event contains minimal data — just what happened and an identifier. Consumers must call back to the source for full details.
```json
{ "type": "OrderPlaced", "orderId": "123" }
```
- **Use when:** Consumers might not need the data, or data is large and varies per consumer. Reduces coupling but adds a network call.

**Event-Carried State Transfer:** The event contains all the data the consumers need.
```json
{ "type": "OrderPlaced", "orderId": "123", "items": [...], "total": 99.99, "customer": { "name": "...", "email": "..." } }
```
- **Use when:** Multiple consumers need the same data (avoids callback storms), you want consumers to be autonomous (can process even if source is down), or you want to reduce latency.

**Tradeoff:** Event-carried state transfer increases event size and coupling to the event schema, but reduces runtime coupling between services.

### Answer 5

**Why event sourcing for everything is bad:**
1. **Complexity overhead** — Simple CRUD operations become unnecessarily complex. A settings page doesn't need an event log.
2. **Storage costs** — Every state change creates a new event. For high-volume, low-value data, this is wasteful.
3. **Query complexity** — You need projections for every read pattern. For simple lookups, this is over-engineered.
4. **Schema evolution** — Evolving event schemas across all services is painful. Changing an event format from 2 years ago affects replays.
5. **Learning curve** — Team must understand eventual consistency, projections, snapshots.

**When event sourcing IS a good choice:**
- **Financial systems** — need complete audit trail (every transaction must be traceable)
- **Collaborative editing** — Google Docs-style concurrent modifications
- **Complex domain processes** — order lifecycle, insurance claims (rich state transitions)
- **Regulatory compliance** — healthcare, banking (must prove what happened and when)
- **Temporal queries** — "What was the account balance on March 15?"

### Answer 6

**Simple Order Flow:**

**Choreography:**
```
Order Service publishes → OrderPlaced
Payment Service consumes OrderPlaced → processes payment → publishes PaymentCompleted
Inventory Service consumes PaymentCompleted → reserves stock → publishes StockReserved
Shipping Service consumes StockReserved → schedules shipment → publishes ShipmentScheduled
Order Service consumes ShipmentScheduled → updates order to "Confirmed"
```

**Orchestration:**
```
Order Saga Orchestrator:
  1. Send ReserveStock command → Inventory Service
  2. Wait for StockReserved response
  3. Send ProcessPayment command → Payment Service
  4. Wait for PaymentCompleted response
  5. Send ScheduleShipment command → Shipping Service
  6. Wait for ShipmentScheduled response
  7. Update order to "Confirmed"
  On failure at any step: send compensating commands
```

**Recommendation:**
- **(a) 3 services** → Choreography. Simple enough to follow the event flow. Less infrastructure needed.
- **(b) 10 services** → Orchestration. With 10 services, choreography becomes impossible to debug. The event flow is scattered across services. An orchestrator provides a single place to see the entire workflow and handle compensation logic.

### Answer 7

**CQRS and Event Sourcing are independent patterns that complement each other:**

- **CQRS without Event Sourcing:** Separate read/write models, but writes store current state (not events). The read model is synchronized via database replication, Change Data Capture, or scheduled sync.

- **Event Sourcing without CQRS:** Store events as the source of truth, but use the same model for reads and writes. Less common because event stores aren't optimized for queries.

- **CQRS + Event Sourcing:** Natural pairing. Events are the write model. Read models (projections) are built by consuming events. This gives you: separate optimization for reads/writes, complete audit trail, ability to rebuild read models.

**You can absolutely use one without the other.** Most CQRS implementations don't use event sourcing.

### Answer 8

For 1TB/day of clickstream data with both real-time dashboards AND monthly reports:

**Recommendation: Kappa Architecture** with caveats.

**Reasoning:**
1. **Single codebase** — One stream processing pipeline reduces maintenance. Clickstream processing logic is the same for real-time and historical.
2. **Modern stream processors** (Kafka + Flink) can handle 1TB/day easily and support both real-time windows and reprocessing.
3. **Monthly reports** — Replay the Kafka topic (with sufficient retention) through the same pipeline to regenerate monthly views.
4. **Simpler operations** — No need to maintain separate batch and speed layers.

**Caveats:**
- Set Kafka retention to cover the reporting period (30+ days) or use tiered storage
- If monthly reports need complex aggregations that are impractical in streaming (e.g., joins across 1-month windows), consider adding a batch layer for just those use cases — making it a hybrid approach

**When Lambda would be better:**
- If the monthly reports require fundamentally different processing logic than real-time
- If you need to join clickstream data with large reference datasets that change monthly
- If the team already has a batch processing infrastructure (Spark) but not stream processing

### Answer 9

Use Web-Queue-Worker instead of microservices when:

1. **Simple separation is enough** — You just need to separate synchronous request handling from async background work. You don't need independent services with separate data stores.
2. **Small team** — One team can manage the web and worker components. No need for service-per-team ownership.
3. **Single domain** — The application doesn't have multiple distinct bounded contexts that would benefit from separate services.
4. **Stepping stone** — You're migrating from a monolith and want to start extracting async work before committing to full microservices.
5. **Cost-sensitive** — Web-Queue-Worker is cheaper to operate than a full microservices platform.

**Example:** An invoice processing system. The web frontend accepts upload requests, queues them, and workers process PDFs asynchronously. One domain, one team, simple separation.

### Answer 10

**"Smart endpoints, dumb pipes"** means:
- **Smart endpoints:** Business logic lives in the services themselves. Each service is responsible for its own processing, validation, transformation, and routing decisions.
- **Dumb pipes:** The communication infrastructure (HTTP, message queue) just transports messages. No transformation, routing logic, or business rules in the middleware.

**Contrast with SOA:**
- SOA uses an Enterprise Service Bus (ESB) — a **smart pipe** that handles message transformation, routing, orchestration, protocol conversion, and sometimes business rules.
- In SOA, significant logic lives in the middleware: XSLT transformations, content-based routing, protocol bridging.

**Why microservices prefer dumb pipes:**
1. The ESB becomes a single point of failure and a bottleneck
2. Logic in the ESB is hard to test and deploy independently
3. The ESB team becomes a bottleneck for all service changes
4. Services are implicitly coupled through the ESB configuration

**In practice:** Use HTTP/REST or gRPC for synchronous communication, and a simple message broker (Kafka, RabbitMQ) for async — no transformation or routing logic in the middleware.

---

## Key Takeaways

1. **There is no best architecture.** Every style is a set of tradeoffs. The architect's job is matching tradeoffs to business needs.
2. **Start simple.** Layered monolith → modular monolith → microservices. Don't jump to microservices on day one.
3. **Combine styles.** Most real systems use multiple architecture styles. An e-commerce platform might use microservices (overall), CQRS (for product catalog), event-driven (for order processing), and Web-Queue-Worker (for report generation).
4. **Architecture drives team structure** (Conway's Law). Microservices need team-per-service. If you can't restructure teams, microservices will struggle.
5. **Event-driven and microservices are natural partners.** Events provide the loose coupling that microservices need.

---

*Previous: [01 - Cloud-Native Fundamentals](01-cloud-native-fundamentals.md)*  
*Next: [03 - Microservices Deep Dive](03-microservices-deep-dive.md)*