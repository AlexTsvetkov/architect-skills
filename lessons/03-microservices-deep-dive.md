# Lesson 03: Microservices Architecture — Deep Dive

> **Prerequisites:** Lesson 01, Lesson 02  
> **Time estimate:** 8-10 hours  
> **Parallel reading:** "Building Microservices" (2nd Ed) — Sam Newman, "DDD Distilled" — Vaughn Vernon

---

## Table of Contents

1. [Domain-Driven Design for Microservices](#1-domain-driven-design-for-microservices)
2. [Decomposition Strategies](#2-decomposition-strategies)
3. [Data Ownership](#3-data-ownership)
4. [Inter-Service Communication](#4-inter-service-communication)
5. [API Gateway Pattern](#5-api-gateway-pattern)
6. [Service Discovery](#6-service-discovery)
7. [Distributed Transactions — Saga Pattern](#7-distributed-transactions--saga-pattern)
8. [Testing Microservices](#8-testing-microservices)
9. [Anti-Patterns](#9-anti-patterns)
10. [Exercises](#10-exercises)
11. [Self-Check Questions](#11-self-check-questions)
12. [Answers](#12-answers)

---

## 1. Domain-Driven Design for Microservices

DDD is the most reliable method for finding microservice boundaries. The key concept is the **Bounded Context**.

### Strategic DDD Concepts

#### Bounded Context

A boundary within which a domain model is consistent and unambiguous. Different bounded contexts can have different models for the same real-world concept.

```
E-COMMERCE DOMAIN

┌─────────────────┐  ┌──────────────────┐  ┌─────────────────┐
│  CATALOG Context │  │  ORDER Context    │  │ SHIPPING Context │
│                  │  │                   │  │                  │
│  Product:        │  │  Product:         │  │  Package:        │
│  - name          │  │  - productId      │  │  - packageId     │
│  - description   │  │  - quantity       │  │  - weight        │
│  - price         │  │  - price (at time │  │  - dimensions    │
│  - categories    │  │    of order)      │  │  - destination   │
│  - images        │  │  - discount       │  │  - trackingNum   │
│                  │  │                   │  │                  │
│  "Product" is    │  │  "Product" means  │  │  "Product" is    │
│  a catalog entry │  │  a line item      │  │  a "Package"     │
└─────────────────┘  └──────────────────┘  └─────────────────┘

Same real-world thing, different models in each context.
This is CORRECT — don't create one universal Product model.
```

#### Ubiquitous Language

Each bounded context has its own vocabulary:

| Term | Catalog Context | Order Context | Shipping Context |
|------|----------------|---------------|-----------------|
| Product | Item for sale with catalog info | Line item with quantity and price snapshot | Physical package with weight/dimensions |
| Customer | Browser with preferences | Buyer with billing info | Recipient with shipping address |
| Price | Current listed price | Price at time of purchase | Declared value for customs |

#### Context Map

Shows how bounded contexts relate to each other:

```
Relationship types:
- U/D = Upstream/Downstream (Catalog provides data, Order consumes)
- ACL = Anti-Corruption Layer (protects from external model changes)
- Partnership = Two contexts evolve together
- Shared Kernel = Shared subset of the model
- Conformist = Downstream adopts upstream's model
- Open Host Service = Well-defined protocol for others to integrate
```

### Tactical DDD Concepts

#### Aggregates

A cluster of domain objects treated as a single unit for data consistency. Each aggregate has a root entity.

```java
// Order is an Aggregate Root
public class Order {
    private OrderId id;
    private CustomerId customerId;       // Reference by ID only
    private List<OrderLine> lines;       // Entities within the aggregate
    private Money totalAmount;           // Value object
    private OrderStatus status;

    public void addItem(ProductId productId, int quantity, Money price) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify submitted order");
        }
        lines.add(new OrderLine(productId, quantity, price));
        recalculateTotal();
    }

    public void submit() {
        if (lines.isEmpty()) {
            throw new IllegalStateException("Cannot submit empty order");
        }
        this.status = OrderStatus.SUBMITTED;
        registerEvent(new OrderSubmitted(this.id, this.totalAmount));
    }
}
```

**Key rule:** Aggregates reference other aggregates by ID only. This enables service boundaries.

### DDD to Microservices Mapping

```
Bounded Context    →  Microservice (or a few closely related services)
Aggregate          →  Unit of consistency within a service
Domain Event       →  Integration event between services
Ubiquitous Lang.   →  API contracts and event schemas
Context Map        →  Service dependency diagram
Anti-Corruption L. →  API adapter/translator between services
```

### Enterprise Monolith Context

In a typical enterprise e-commerce monolith, modules map roughly to bounded contexts:
- Catalog module → Catalog context
- Order module → Order context
- Payment module → Payment context
- Warehouse/Fulfillment module → Inventory/Shipping context

**But** in a monolith, these modules typically share a single database and a common data model. In microservices, each service owns its data and communicates via APIs/events. The transition from shared database to per-service data ownership is one of the hardest challenges in decomposition.

---

## 2. Decomposition Strategies

### Strategy 1: Decompose by Business Capability

```
Business Capabilities:
├── Product Management → Catalog Service, Pricing Service
├── Order Management   → Order Service, Returns Service
├── Customer Mgmt      → Customer Service, Loyalty Service
├── Payment            → Payment Service, Refund Service
└── Shipping           → Shipping Service, Tracking Service
```

### Strategy 2: Decompose by Subdomain (DDD)

```
Core Domain (competitive advantage — invest most):
├── Recommendation Engine
├── Pricing Strategy
└── Customer Experience

Supporting Domain (necessary but not differentiating):
├── Order Management
├── Inventory Management
└── Shipping

Generic Domain (commodity — buy or use SaaS):
├── Authentication (Keycloak/Auth0)
├── Payment (Stripe)
├── Email (SendGrid)
└── Monitoring (Datadog)
```

### Strategy 3: Strangler Fig Pattern (Migration)

```
Phase 1: Route specific requests to new service
Client → Router → Monolith (default)
                 → Product Service (/products only)

Phase 2: Gradually extract more
Client → Router → Monolith (shrinking)
                 → Product Service
                 → Order Service
                 → Payment Service

Phase 3: Monolith fully replaced
Client → API Gateway → Product Service
                     → Order Service
                     → Payment Service
```

### How to Identify Service Boundaries

1. **Does it have its own data?** If two services share a table, they're likely one service.
2. **Can it be deployed independently?** If deploying A always requires B, they're one service.
3. **Does it have a clear API?** If the interface is vague, the boundary is wrong.
4. **Does a single team own it?** One team per service (a team can own multiple).
5. **Does it match a business capability?** Align with business, not technical layers.

---

## 3. Data Ownership

### Database-per-Service Pattern

```
SHARED DATABASE (Anti-pattern):          DATABASE-PER-SERVICE (Target):
┌────────┐ ┌────────┐ ┌────────┐         ┌────────┐ ┌────────┐ ┌────────┐
│Order   │ │Payment │ │Ship    │         │Order   │ │Payment │ │Ship    │
│Service │ │Service │ │Service │         │Service │ │Service │ │Service │
└───┬────┘ └───┬────┘ └───┬────┘         └───┬────┘ └───┬────┘ └───┬────┘
    │          │          │                   │          │          │
    └──────────┴──────────┘               ┌───┴──┐  ┌───┴──┐  ┌───┴──┐
              │                           │OrderDB│  │PayDB │  │ShipDB│
        ┌─────┴─────┐                    │Postgres│  │Postgres│  │MongoDB│
        │ Shared DB  │                    └───────┘  └───────┘  └───────┘
        └───────────┘
```

### Handling Cross-Service Queries

**Problem:** "Show order #123 with customer name, payment status, and shipping tracking."

#### Solution 1: API Composition

```
API Gateway / Aggregator:
  1. GET /orders/123        → { orderId, customerId, items }
  2. GET /customers/{id}    → { name, email }
  3. GET /payments?order=123 → { status, amount }
  4. GET /shipments?order=123 → { tracking, eta }
  → Compose into single response
```

#### Solution 2: CQRS with Read Model

```
Services publish events → Event Store / Kafka
                            → Projector consumes events
                            → Denormalized Read Model (Elasticsearch)
                            → Query the read model directly
```

#### Solution 3: Data Replication via Events

```
Customer Service publishes CustomerUpdated event
Order Service consumes it → stores { customerId, name } locally
Now Order Service answers "order + customer name" without calling Customer Service
```

**Tradeoff:** Denormalization + eventual consistency vs. runtime coupling.

---

## 4. Inter-Service Communication

### Synchronous: REST vs. gRPC

| Aspect | REST (HTTP/JSON) | gRPC (HTTP/2 + Protobuf) |
|--------|-----------------|--------------------------|
| Speed | Standard | 10x faster (binary, HTTP/2) |
| Typing | Loose (JSON) | Strong (Protobuf schema) |
| Streaming | No native support | Bidirectional streaming |
| Browser support | Native | Limited (needs grpc-web) |
| Human readable | Yes | No |
| Best for | Public APIs, CRUD | Internal service-to-service, high-perf |

### Asynchronous: Commands vs. Events

| Pattern | Commands | Events |
|---------|----------|--------|
| Direction | Targeted: "Do this" | Broadcast: "This happened" |
| Consumers | One specific consumer | Any number of consumers |
| Coupling | Producer knows consumer | Producer doesn't know consumers |
| Example | `ProcessPayment { orderId }` | `OrderPlaced { orderId, items }` |

### The Synchronous Chain Problem

```
❌ Client → A → B → C → D → E
  Latency = sum of all latencies
  Availability = product of all availabilities (99.9%^5 = 99.5%)

✅ Client → A → publishes event
  B, C, D, E consume events asynchronously
  Much more resilient and scalable
```

---

## 5. API Gateway Pattern

```
┌────────┐     ┌────────────────────────────────┐
│ Clients│────→│         API GATEWAY             │
└────────┘     │ • Authentication                │
               │ • Rate Limiting                 │
               │ • SSL Termination               │
               │ • Request Routing               │
               │ • Response Aggregation          │
               │ • Circuit Breaking              │
               │ • Logging / Monitoring          │
               └──────┬──────┬──────┬────────────┘
                      │      │      │
                   Svc A  Svc B  Svc C
```

### Backend for Frontend (BFF) Pattern

```
Mobile App → Mobile BFF  → Services (optimized payloads, fewer calls)
Web App    → Web BFF     → Services (richer responses, SSR)
3rd Party  → Public API  → Services (versioned, rate-limited)
```

### API Gateway Technologies

| Technology | Type | Best For |
|-----------|------|----------|
| Kong | Open-source | Self-managed, extensible |
| AWS API Gateway | Managed | AWS ecosystem, serverless |
| Azure API Management | Managed | Azure ecosystem |
| Spring Cloud Gateway | Java-native | Spring ecosystem |
| Envoy / Istio | Service proxy | Service mesh |

---

## 6. Service Discovery

### Client-Side Discovery

```
Order Service → asks Registry "where is Payment Service?"
             ← gets list: [payment:8081, payment:8082, payment:8083]
             → chooses one (client-side load balancing)
             → calls payment:8082 directly

Tools: Netflix Eureka, Consul, etcd
```

### Server-Side Discovery

```
Order Service → calls "payment-service" (logical name)
             → DNS/Load Balancer resolves to actual instance
             → request routed to payment:8082

Tools: Kubernetes Services (built-in!), AWS ALB
```

### Kubernetes Service Discovery (Built-in)

```yaml
# Kubernetes Service — provides DNS-based discovery
apiVersion: v1
kind: Service
metadata:
  name: payment-service
spec:
  selector:
    app: payment
  ports:
    - port: 80
      targetPort: 8080

# Other services call: http://payment-service/api/payments
# Kubernetes DNS resolves "payment-service" to pod IPs
# Built-in load balancing across pods
```

In Kubernetes, you rarely need external service discovery. Kubernetes Services handle it.

---

## 7. Distributed Transactions — Saga Pattern

### Why We Need Sagas

In a monolith, you use ACID transactions:
```sql
BEGIN TRANSACTION;
  INSERT INTO orders (...);
  UPDATE inventory SET qty = qty - 1;
  INSERT INTO payments (...);
COMMIT;  -- all or nothing
```

In microservices, each service has its own database. **No distributed ACID transactions.** Instead, use the Saga pattern.

### Orchestration-Based Saga

```
┌──────────────────────────────────────────────────────┐
│                 ORDER SAGA ORCHESTRATOR               │
│                                                      │
│  Step 1: Create Order (PENDING)                      │
│    → Order Service                                   │
│                                                      │
│  Step 2: Reserve Inventory                           │
│    → Inventory Service                               │
│    ✗ Compensation: Cancel Order                      │
│                                                      │
│  Step 3: Process Payment                             │
│    → Payment Service                                 │
│    ✗ Compensation: Release Inventory, Cancel Order   │
│                                                      │
│  Step 4: Schedule Shipping                           │
│    → Shipping Service                                │
│    ✗ Compensation: Refund, Release Inventory, Cancel │
│                                                      │
│  Step 5: Confirm Order (CONFIRMED)                   │
│    → Order Service                                   │
└──────────────────────────────────────────────────────┘
```

### Choreography-Based Saga

```
Order Service:      OrderCreated ──→
Inventory Service:  ←── consumes, reserves stock ──→ StockReserved
Payment Service:    ←── consumes, charges ──→ PaymentProcessed
Shipping Service:   ←── consumes, schedules ──→ ShipmentScheduled
Order Service:      ←── consumes, confirms order

On failure (e.g., payment fails):
Payment Service: ──→ PaymentFailed
Inventory Service: ←── consumes ──→ StockReleased
Order Service: ←── consumes ──→ OrderCancelled
```

### When to Choose Each

| Aspect | Orchestration | Choreography |
|--------|--------------|--------------|
| Complexity of flow | Complex (5+ steps, conditions) | Simple (3-4 linear steps) |
| Visibility | Easy — single orchestrator | Hard — flow across services |
| Coupling | Services coupled to orchestrator | Services loosely coupled |
| Adding steps | Modify orchestrator | Add new event consumer |
| Error handling | Centralized compensation | Distributed compensation |

---

## 8. Testing Microservices

### The Testing Pyramid for Microservices

```
              ┌───────────┐
              │   E2E     │  Few — slow, expensive, brittle
              │   Tests   │  Test critical business flows
              ├───────────┤
              │ Contract  │  Medium — validate API agreements
              │   Tests   │  Consumer-driven (Pact)
              ├───────────┤
              │Integration│  Medium — test with real dependencies
              │   Tests   │  Testcontainers for DB, Kafka, etc.
              ├───────────┤
              │   Unit    │  Many — fast, isolated
              │   Tests   │  Domain logic, business rules
              └───────────┘
```

### Contract Testing (Pact)

```
Consumer (Order Service) defines:
  "When I call GET /inventory/PROD-123,
   I expect { productId: string, available: number }"

Provider (Inventory Service) verifies:
  "My API satisfies the contract from Order Service"

If a provider change breaks a consumer contract → build fails
No need for integration environment to catch API mismatches
```

### Integration Testing with Testcontainers

```java
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.5.0"));

    @Test
    void shouldCreateOrderAndPublishEvent() {
        // Test with real PostgreSQL and Kafka
        // No mocks — real integration
    }
}
```

### Chaos Engineering

Deliberately inject failures to test resilience:
- **Kill random pods** — Does the system recover?
- **Add network latency** — Do circuit breakers trigger?
- **Fill disk** — Do log rotation and alerts work?
- **Simulate DNS failures** — Does service discovery fail gracefully?

Tools: Chaos Monkey, Litmus Chaos, Gremlin

---

## 9. Anti-Patterns

### 1. Distributed Monolith

```
❌ "Microservices" that must be deployed together
❌ Services that share a database
❌ Synchronous chains for every operation
❌ One team owns all services

Result: All the complexity of distributed systems
        with none of the benefits of microservices
```

### 2. Nano-Services (Too Small)

```
❌ UserNameService, UserEmailService, UserAddressService
   → Should be a single UserService

Signs you're too granular:
- Services have 1-2 endpoints
- Most changes require modifying multiple services
- Excessive inter-service communication for simple operations
```

### 3. Shared Database

```
❌ Multiple services reading/writing the same tables
   → Tight coupling through the database schema
   → Cannot change schema without coordinating all services
   → Cannot use different DB technologies per service
```

### 4. No API Versioning

```
❌ Changing API contracts without versioning
   → Breaking all consumers on every deployment

✅ Version your APIs:
   /api/v1/orders  — current
   /api/v2/orders  — new format (deprecate v1 later)
```

### 5. Chatty Communication

```
❌ 20 REST calls to render one page
   → Use aggregation, BFF, or event-carried state transfer
   → Batch endpoints: POST /api/batch
```

---

## 10. Exercises

### Exercise 1: Domain Decomposition

Take an e-commerce platform and identify:
1. At least 6 bounded contexts using DDD strategic design
2. Draw a context map showing relationships (U/D, ACL, Partnership)
3. Identify aggregates within two of the contexts
4. Define domain events that flow between contexts
5. Map bounded contexts to microservices

### Exercise 2: Saga Design

Design a saga for a hotel booking system:
- Reserve room
- Charge payment
- Send confirmation email
- Update loyalty points
- Assign room number

Design both orchestration and choreography approaches. Include:
- Happy path flow
- Failure handling (what if payment fails? what if room becomes unavailable?)
- Compensating transactions
- Which approach would you recommend and why?

### Exercise 3: Data Ownership Challenge

You're decomposing a monolith e-commerce database. The `orders` table currently JOINs with `customers`, `products`, `payments`, and `addresses`.

Design the data ownership for microservices:
1. Which service owns which data?
2. How do you handle the current JOIN queries?
3. What data needs to be replicated and how?
4. How do you ensure data consistency across services?

### Exercise 4: Monolith Decomposition

You are tasked with decomposing a large Spring Boot e-commerce monolith (500K+ LOC) into microservices. The monolith has these modules: catalog, order, payment, shipping, promotions, customer, and inventory — all sharing a single PostgreSQL database with 200+ tables and heavy use of foreign keys across module boundaries.

Design a microservices decomposition:
1. Map monolith modules to bounded contexts using domain analysis
2. Identify which modules should become separate services vs. stay together (justify with coupling analysis)
3. Design the strangler fig migration plan (what to extract first and why — consider risk, coupling, and business value)
4. Address the shared database challenge: how do you break foreign key dependencies and maintain data consistency across services?

---

## 11. Self-Check Questions

1. What is a bounded context and why is it the best guide for microservice boundaries?

2. Explain the difference between aggregates, entities, and value objects in DDD.

3. Why is the "database per service" pattern important? What problems does it solve and create?

4. When would you choose gRPC over REST for inter-service communication?

5. What is the distributed monolith anti-pattern? How do you detect it?

6. Describe the saga pattern. When would you use orchestration vs. choreography?

7. What is contract testing and why is it essential for microservices?

8. A team wants to decompose their monolith into 30 microservices on day one. What advice would you give?

9. How does the strangler fig pattern work? Why is it preferred over a big-bang rewrite?

10. What is the BFF pattern and when would you use it?

---

## 12. Answers

### Answer 1

A **bounded context** is a boundary within which a particular domain model applies consistently. It defines the scope within which a ubiquitous language has a specific meaning.

It's the best guide for microservice boundaries because:
1. **Aligns with business domains** — not technical layers
2. **Encapsulates a consistent model** — the "Product" in catalog is different from "Product" in orders, and that's ok
3. **Minimizes cross-boundary communication** — most operations happen within one context
4. **Enables team ownership** — one team per bounded context
5. **Prevents the "god model" problem** — no universal data model shared by all services

### Answer 2

- **Aggregate:** A cluster of domain objects that are treated as a single unit for data changes. It has a root entity and enforces invariants. Example: `Order` aggregate containing `OrderLine` entities. All changes to order lines go through the `Order` root.

- **Entity:** An object with a distinct identity that persists over time. Two entities with the same attributes are still different if they have different IDs. Example: `Customer` (ID: C-123 is always the same customer, even if they change their name).

- **Value Object:** An object defined by its attributes, not its identity. Two value objects with the same attributes are equal. Immutable. Example: `Money(100, "USD")` — two instances with the same amount and currency are interchangeable.

### Answer 3

**Why database-per-service matters:**
- **Independent schema evolution** — change your schema without coordinating with other teams
- **Technology freedom** — use PostgreSQL for orders, MongoDB for catalog, Redis for sessions
- **Independent scaling** — scale your database based on your service's needs
- **Loose coupling** — no hidden coupling through shared tables

**Problems it creates:**
- **Cross-service queries** — can't JOIN across databases; need API composition, CQRS, or data replication
- **Distributed transactions** — can't use ACID transactions across services; need sagas
- **Data consistency** — eventual consistency instead of strong consistency
- **Data duplication** — some data may be replicated across services

### Answer 4

Choose gRPC over REST when:
1. **Internal service-to-service communication** — where browser compatibility doesn't matter
2. **High performance requirements** — gRPC is ~10x faster due to binary serialization and HTTP/2
3. **Streaming data** — gRPC supports bidirectional streaming natively
4. **Polyglot environments** — Protobuf generates client/server code for multiple languages
5. **Strong API contracts** — Protobuf schema provides strict typing and backward compatibility

Stay with REST for: public APIs, browser clients, simple CRUD, when human readability matters.

### Answer 5

**Distributed Monolith:** A system that looks like microservices but has all the drawbacks of a monolith plus all the complexity of a distributed system.

**Detection signs:**
1. Services must be deployed together (deploying A requires deploying B)
2. Services share a database
3. A change in one service requires changes in multiple other services
4. Long synchronous call chains
5. One team manages all services
6. Services share libraries that contain business logic
7. No independent testing — services can only be tested together

**Root causes:** Premature decomposition, shared database, lack of DDD, insufficient API contracts.

### Answer 6

**Saga Pattern:** Manages distributed transactions as a sequence of local transactions. Each step has a compensating action for rollback.

**Orchestration:** A central coordinator (saga orchestrator) directs each step, waits for results, and triggers compensations on failure. Like a conductor leading an orchestra.
- **Use when:** Complex flows (5+ steps), conditional logic, visibility is important, centralized error handling needed.

**Choreography:** Each service reacts to events and publishes its own events. No central coordinator. Like dancers who know their routines and react to each other.
- **Use when:** Simple linear flows (3-4 steps), high autonomy, loose coupling is priority, services are managed by different teams.

### Answer 7

**Contract Testing** validates that the API contract between a consumer and provider is maintained. The consumer defines what it expects; the provider verifies it satisfies all consumer contracts.

**Why essential for microservices:**
1. **Catches breaking changes early** — before they reach production
2. **No integration environment needed** — tests run in CI/CD pipelines
3. **Independent deployability** — services can deploy confidently knowing they won't break consumers
4. **Documentation** — contracts serve as living documentation of API agreements
5. **Scales** — as services multiply, contract tests prevent combinatorial integration testing

Tool: **Pact** is the most widely used framework.

### Answer 8

Advice for the team wanting 30 microservices on day one:

1. **Don't.** Start with a modular monolith or 3-5 coarse-grained services.
2. **You don't understand the boundaries yet.** Wrong boundaries lead to a distributed monolith. Discover boundaries through a modular monolith first.
3. **You're not ready operationally.** 30 services need: container orchestration, CI/CD per service, distributed tracing, centralized logging, service mesh. Build this infrastructure first.
4. **Start with the strangler fig.** Extract one service at a time. Prove the pattern works. Then accelerate.
5. **Follow the "two-pizza team" rule.** You need at least 5-6 teams (30 services ÷ 5-6 per team) with true ownership. If you have 2 teams, you can't manage 30 services.
6. **Extract based on pain.** Start with the component that causes the most deployment friction or scaling issues.

### Answer 9

**Strangler Fig Pattern:** Named after strangler fig trees that grow around existing trees.

**How it works:**
1. Put a routing layer (API gateway) in front of the monolith
2. Implement new functionality in a new service
3. Route requests for that functionality to the new service
4. The monolith gradually shrinks as more functionality is extracted
5. Eventually, the monolith is fully replaced

**Why preferred over big-bang rewrite:**
1. **Lower risk** — incremental changes, not all-or-nothing
2. **Continuous delivery of value** — ship new services while monolith still works
3. **Learn as you go** — discover service boundaries through practice
4. **Rollback is easy** — if a new service fails, route back to monolith
5. **No "feature freeze"** — the monolith continues to evolve during migration

### Answer 10

**BFF (Backend for Frontend):** A separate backend service tailored for each frontend type (mobile, web, 3rd party).

**When to use:**
- **Mobile vs. web have different needs** — mobile needs smaller payloads, fewer round trips; web needs richer data
- **Different teams own different frontends** — mobile team controls their BFF, web team controls theirs
- **API aggregation** — BFF composes multiple microservice calls into one frontend-friendly response
- **Different auth flows** — mobile uses OAuth with refresh tokens; web uses session cookies
- **Different performance profiles** — mobile needs aggressive caching; web needs real-time data

**When NOT to use:**
- Single frontend — one API gateway is sufficient
- All frontends need the same data — BFF adds unnecessary duplication

---

## Key Takeaways

1. **Use DDD to find service boundaries.** Bounded contexts are the best guide.
2. **Each service owns its data.** No shared databases. This is non-negotiable.
3. **Prefer async communication.** Synchronous chains kill availability and performance.
4. **Start with the modular monolith.** Extract services when boundaries are proven.
5. **Saga pattern replaces ACID transactions.** Choose orchestration for complex flows, choreography for simple ones.
6. **Contract testing enables independent deployment.** Without it, microservices become a coordination nightmare.
7. **Avoid anti-patterns.** Distributed monolith, nano-services, and shared databases defeat the purpose.

---

*Previous: [02 - Architecture Styles](02-architecture-styles.md)*  
*Next: [04 - Distributed Systems & Data](04-distributed-systems-data.md)*