# Lesson 03: Microservices Architecture — Deep Dive

> **Prerequisites:** Lesson 01, 02  
> **Time estimate:** 6-8 hours  
> **Parallel reading:** "Building Microservices" (Ch 1-6) + "DDD Distilled" — Vaughn Vernon

---

## Table of Contents

1. [Domain-Driven Design (DDD)](#1-domain-driven-design-ddd)
2. [Decomposition Strategies](#2-decomposition-strategies)
3. [Data Ownership](#3-data-ownership)
4. [Inter-Service Communication](#4-inter-service-communication)
5. [API Gateway & BFF Patterns](#5-api-gateway--bff-patterns)
6. [Service Discovery](#6-service-discovery)
7. [Distributed Transactions — Saga Pattern](#7-distributed-transactions--saga-pattern)
8. [Testing Microservices](#8-testing-microservices)
9. [Anti-Patterns](#9-anti-patterns)
10. [Exercises](#10-exercises)
11. [Self-Check Questions](#11-self-check-questions)
12. [Answers](#12-answers)

---

## 1. Domain-Driven Design (DDD)

### Strategic DDD — Finding Service Boundaries

DDD is the primary tool for identifying correct microservice boundaries. The core concepts:

**Bounded Context** — A boundary within which a particular domain model is defined and applicable. Each bounded context has its own ubiquitous language (the same word can mean different things in different contexts).

```
E-COMMERCE BOUNDED CONTEXTS:
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   CATALOG   │    │   ORDERING  │    │   SHIPPING  │     │
│  │             │    │             │    │             │     │
│  │ "Product"   │    │ "Product"   │    │ "Product"   │     │
│  │ = SKU, name,│    │ = line item,│    │ = package   │     │
│  │   desc,     │    │   quantity, │    │   dimensions│     │
│  │   images,   │    │   price     │    │   weight,   │     │
│  │   reviews   │    │             │    │   fragility │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                              │
│  Same word "Product" — different meaning in each context     │
│  Each context has its OWN model of Product                   │
└─────────────────────────────────────────────────────────────┘
```

**Context Map** — Shows relationships between bounded contexts:

```
┌──────────┐        ┌──────────┐         ┌──────────┐
│ Catalog  │───U/D──│ Ordering │──U/D────│ Shipping │
│ Context  │        │ Context  │         │ Context  │
└──────────┘        └────┬─────┘         └──────────┘
                         │
                    ┌────┴─────┐
                    │ Payment  │──ACL──→ External
                    │ Context  │         Payment
                    └──────────┘         Gateway

Relationship Types:
U/D = Upstream/Downstream (upstream publishes, downstream consumes)
ACL = Anti-Corruption Layer (translation between models)
OHS = Open Host Service (public API for multiple consumers)
```

### Tactical DDD — Building Blocks

```java
// ENTITY — has identity, mutable state
@Entity
public class Order {
    @Id
    private OrderId id;          // Identity — two orders with same data are NOT the same order
    private OrderStatus status;
    private List<OrderLine> lines;
    private Money totalAmount;
    
    // Business logic lives IN the entity
    public void addItem(Product product, int quantity) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify a submitted order");
        }
        lines.add(new OrderLine(product.getSku(), product.getPrice(), quantity));
        recalculateTotal();
    }
    
    public void submit() {
        if (lines.isEmpty()) throw new IllegalStateException("Cannot submit empty order");
        this.status = OrderStatus.SUBMITTED;
        registerEvent(new OrderSubmittedEvent(this.id, this.totalAmount));
    }
}

// VALUE OBJECT — no identity, immutable, defined by attributes
public record Money(BigDecimal amount, Currency currency) {
    public Money {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        Objects.requireNonNull(currency, "Currency is required");
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
}

// AGGREGATE — cluster of entities and value objects with a root entity
// The aggregate root is the ONLY entry point for modifications
// All invariants are enforced by the aggregate root
public class Order { // This IS the aggregate root
    // OrderLine is part of the Order aggregate — cannot be accessed independently
    // Address is a value object within the aggregate
    // Order enforces all business rules (min items, max total, valid transitions)
}
```

**Aggregate Rules:**
1. Reference other aggregates by ID only (not by object reference)
2. One transaction = one aggregate (don't modify multiple aggregates in one transaction)
3. Keep aggregates small — large aggregates cause contention

---

## 2. Decomposition Strategies

### Strategy 1: By Business Capability

```
Business Capabilities (what the business DOES):
├── Product Management    → Catalog Service
├── Order Management      → Order Service
├── Payment Processing    → Payment Service
├── Inventory Management  → Inventory Service
├── Shipping & Delivery   → Shipping Service
├── Customer Management   → Customer Service
└── Notifications         → Notification Service
```

### Strategy 2: By Subdomain (DDD)

```
Core Domain (competitive advantage — build yourself):
├── Recommendation Engine
├── Dynamic Pricing
└── Personalization

Supporting Domain (necessary but not differentiating — build or buy):
├── Order Management
├── Inventory Management
└── Customer Management

Generic Domain (commodity — buy or use SaaS):
├── Payment Processing → Stripe, PayPal
├── Email/SMS          → SendGrid, Twilio
├── Authentication     → Keycloak, Auth0
└── Search             → Elasticsearch (managed)
```

### Strategy 3: Strangler Fig Pattern (Migration)

```
PHASE 1: Intercept                PHASE 2: Extract               PHASE 3: Retire
┌────────┐                       ┌────────┐                      ┌────────┐
│  API   │                       │  API   │                      │  API   │
│Gateway │                       │Gateway │                      │Gateway │
└───┬────┘                       └───┬────┘                      └───┬────┘
    │                                │                               │
    ├──→ Monolith (all traffic)      ├──→ Monolith (most traffic)    ├──→ Order Service
    │                                │                               ├──→ Payment Service
    │                                ├──→ Order Service (new)        ├──→ Catalog Service
    │                                │    (extracted)                │
    │                                │                               │    Monolith
    │                                │                               │    (retired)

Timeline: Months to years — incremental, low-risk migration
```

---

## 3. Data Ownership

### Database-Per-Service Pattern

```
┌─────────────────────────────────────────────────┐
│                    BEFORE (Monolith)              │
│  ┌───────────────────────────────────────────┐   │
│  │           Shared Database                  │   │
│  │  ┌────────┬─────────┬──────────┬────────┐ │   │
│  │  │orders  │customers│ products │payments│ │   │
│  │  └────────┴─────────┴──────────┴────────┘ │   │
│  └───────────────────────────────────────────┘   │
│  Easy JOINs across all tables                     │
│  Single transaction for cross-domain changes      │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│                    AFTER (Microservices)          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │Order DB  │  │Customer  │  │Product DB│      │
│  │(Postgres)│  │DB (Postgres)│ │(MongoDB) │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│  No cross-service JOINs!                         │
│  No cross-service transactions!                  │
│  Each service has FULL control over its data      │
└─────────────────────────────────────────────────┘
```

### Solving Cross-Service Queries

| Pattern | How | When | Example |
|---------|-----|------|---------|
| **API Composition** | Aggregate data from multiple services at the API layer | Simple queries, few services | Order details page: call Order + Customer + Product APIs |
| **CQRS with Events** | Build denormalized read models from events | Complex queries, high read volume | Dashboard showing order + customer + product info |
| **Data Replication** | Services subscribe to events and store local copies of needed data | Frequent cross-service lookups | Order service stores customer name locally |

---

## 4. Inter-Service Communication

### Synchronous vs. Asynchronous

```
SYNCHRONOUS (request/response):
Order Service ──HTTP/gRPC──→ Payment Service ──HTTP/gRPC──→ Fraud Service
     │                            │                            │
     │←────── response ───────────│←────── response ──────────│
     
⚠️ THE CHAIN PROBLEM:
Availability = 99.9% × 99.9% × 99.9% = 99.7% (not 99.9%!)
Each service in the chain reduces overall availability.
3 services at 99.9% each = 8.76 hours downtime/year (not 0.876 hours)

ASYNCHRONOUS (event-driven):
Order Service ──event──→ [Kafka] ──event──→ Payment Service
                                  ──event──→ Notification Service
                                  ──event──→ Analytics Service

✅ Services are decoupled — adding consumers is easy
✅ Producer doesn't wait for consumer — non-blocking
✅ Events can be replayed if a consumer was down
```

### REST vs. gRPC Decision Guide

| Factor | REST (HTTP/JSON) | gRPC (HTTP/2 + Protobuf) |
|--------|-----------------|-------------------------|
| **Performance** | Text-based, larger payloads | Binary, 7-10x faster serialization |
| **Contract** | OpenAPI (optional, can drift) | Protocol Buffers (mandatory, strict) |
| **Streaming** | Limited (WebSocket, SSE) | Bidirectional streaming built-in |
| **Browser support** | Native | Requires gRPC-Web proxy |
| **Tooling** | Excellent (Postman, curl, browser) | Growing (grpcurl, BloomRPC) |
| **Best for** | Public APIs, external clients, CRUD | Internal service-to-service, high throughput |

---

## 5. API Gateway & BFF Patterns

### API Gateway

```
┌─────────┐  ┌──────────┐  ┌──────────┐
│ Mobile  │  │   Web    │  │ Partner  │
│  App    │  │  App     │  │  API     │
└────┬────┘  └────┬─────┘  └────┬─────┘
     │            │             │
     └────────────┼─────────────┘
                  │
          ┌───────┴───────┐
          │  API GATEWAY  │
          │               │
          │ • Routing     │
          │ • Auth/AuthZ  │
          │ • Rate limit  │
          │ • SSL term    │
          │ • Monitoring  │
          └───┬───┬───┬───┘
              │   │   │
     ┌────────┘   │   └────────┐
     ▼            ▼            ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Order   │ │ Product │ │ Payment │
│ Service │ │ Service │ │ Service │
└─────────┘ └─────────┘ └─────────┘
```

### Backend for Frontend (BFF)

```
┌─────────┐    ┌──────────┐    ┌──────────┐
│ Mobile  │    │   Web    │    │  Admin   │
│  App    │    │  App     │    │  Panel   │
└────┬────┘    └────┬─────┘    └────┬─────┘
     │              │               │
┌────┴─────┐  ┌────┴──────┐  ┌────┴──────┐
│Mobile BFF│  │  Web BFF  │  │ Admin BFF │  ← Each client type gets
│(optimized│  │(optimized │  │(optimized │     its own backend
│for mobile│  │for web    │  │for admin  │     aggregation layer
│payloads) │  │payloads)  │  │payloads)  │
└────┬─────┘  └────┬──────┘  └────┬──────┘
     │              │               │
     └──────────────┼───────────────┘
                    │
           Shared Microservices
```

---

## 6. Service Discovery

```
CLIENT-SIDE DISCOVERY:                    SERVER-SIDE DISCOVERY:
┌──────────┐                              ┌──────────┐
│  Client  │──→ Registry ──→ Pick one     │  Client  │──→ Load Balancer ──→ Service
│          │        │                     │          │         │
│  (knows  │    ┌───┴────┐                │ (doesn't │    ┌────┴────┐
│   about  │    │Service │                │  care    │    │Registry │
│  registry)│   │Registry│                │  about   │    └─────────┘
└──────────┘    │(Eureka,│                │ discovery)│
                │Consul) │                └──────────┘
                └────────┘

KUBERNETES (built-in server-side):
• DNS-based discovery: http://order-service.default.svc.cluster.local
• Kubernetes Services provide load balancing automatically
• No additional service discovery tool needed
• ✅ Recommended for Kubernetes-native applications
```

---

## 7. Distributed Transactions — Saga Pattern

### The Problem

Microservices don't share databases, so traditional ACID transactions across services don't work. The Saga pattern replaces a single distributed transaction with a sequence of local transactions, each with a compensating action.

### Orchestration Saga Example

```
┌─────────────────────────────────────────────────────────┐
│                  ORDER SAGA ORCHESTRATOR                  │
├─────────────────────────────────────────────────────────┤
│ Step 1: Create Order (PENDING)                           │
│ Step 2: Reserve Inventory      ← compensate: Release     │
│ Step 3: Process Payment        ← compensate: Refund      │
│ Step 4: Confirm Order (CONFIRMED)                        │
│ Step 5: Schedule Shipping      ← compensate: Cancel      │
└─────────────────────────────────────────────────────────┘

Happy Path:
  CreateOrder → ReserveInventory → ProcessPayment → ConfirmOrder → ScheduleShipping ✅

Payment Fails (Step 3):
  CreateOrder → ReserveInventory → ProcessPayment ✗
                                         │
                     ┌───────────────────┘ (compensation starts)
                     ▼
              ReleaseInventory → RejectOrder ✅ (order cancelled cleanly)
```

```java
// Saga Orchestrator (simplified)
@Service
public class OrderSagaOrchestrator {
    
    public void execute(CreateOrderCommand command) {
        OrderId orderId = orderService.createOrder(command);
        
        try {
            inventoryService.reserveItems(orderId, command.getItems());
        } catch (InsufficientStockException e) {
            orderService.rejectOrder(orderId, "Insufficient stock");
            return;
        }
        
        try {
            paymentService.processPayment(orderId, command.getPaymentDetails());
        } catch (PaymentFailedException e) {
            // COMPENSATE: release inventory
            inventoryService.releaseReservation(orderId);
            orderService.rejectOrder(orderId, "Payment failed");
            return;
        }
        
        orderService.confirmOrder(orderId);
        shippingService.scheduleShipping(orderId);
    }
}
```

---

## 8. Testing Microservices

### Testing Pyramid for Microservices

```
          ╱╲
         ╱  ╲        End-to-End Tests (few, slow, expensive)
        ╱ E2E╲       Full system integration across services
       ╱──────╲
      ╱        ╲     Contract Tests (moderate)
     ╱ Contract ╲    Verify API contracts between services (Pact)
    ╱────────────╲
   ╱              ╲   Integration Tests (moderate)
  ╱  Integration   ╲  Service + database + external deps (Testcontainers)
 ╱──────────────────╲
╱                    ╲ Unit Tests (many, fast, cheap)
╱      Unit Tests     ╲ Domain logic, business rules
╱────────────────────────╲
```

### Consumer-Driven Contract Testing (Pact)

```java
// CONSUMER side — defines expected API behavior
@Pact(consumer = "OrderService", provider = "PaymentService")
public V4Pact createPact(PactDslWithProvider builder) {
    return builder
        .given("a valid payment method exists")
        .uponReceiving("a payment processing request")
        .path("/api/payments")
        .method("POST")
        .body(new PactDslJsonBody()
            .stringValue("orderId", "ORD-123")
            .decimalType("amount", 99.99))
        .willRespondWith()
        .status(200)
        .body(new PactDslJsonBody()
            .stringType("transactionId")
            .stringValue("status", "COMPLETED"))
        .toPact(V4Pact.class);
}

// PROVIDER side — verifies it meets the contract
@Provider("PaymentService")
@PactBroker
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PaymentServicePactVerification {
    
    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verifyPact(PactVerificationContext context) {
        context.verifyInteraction();
    }
}
```

### Integration Testing with Testcontainers

```java
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.6.0"));

    @DynamicPropertySource
    static void overrideProps(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }

    @Test
    void shouldCreateOrderAndPublishEvent() {
        // Test with REAL PostgreSQL and REAL Kafka — not mocks
    }
}
```

---

## 9. Anti-Patterns

| Anti-Pattern | Description | Why It's Bad | Fix |
|-------------|-------------|-------------|-----|
| **Distributed Monolith** | Services that must be deployed together | You have the complexity of microservices with none of the benefits | Ensure services can be deployed independently; remove synchronous chains |
| **Shared Database** | Multiple services reading/writing the same tables | Tight coupling — schema changes break multiple services | Database per service; use events for data sharing |
| **Nano-services** | Services that are too small (single function) | Operational overhead exceeds benefit; too many network calls | Merge into larger bounded-context services |
| **God Service** | One service that does too much / orchestrates everything | Single point of failure, deployment bottleneck | Decompose by bounded context; use choreography |
| **Chatty Services** | Services making many fine-grained calls to each other | High latency, network fragility | Coarser-grained APIs; event-carried state transfer |

---

## 10. Exercises

### Exercise 1: Domain Decomposition

Given an e-commerce platform with these features: product catalog, search, user accounts, shopping cart, order placement, payment, inventory, shipping, reviews, recommendations, promotions, notifications.

1. Identify 6-8 bounded contexts
2. Draw a context map with relationship types (U/D, ACL, OHS)
3. For each context, classify as Core, Supporting, or Generic domain
4. Identify which contexts should be custom-built vs. bought/SaaS

### Exercise 2: Saga Design

Design a hotel booking saga with these services: Booking, Room Reservation, Payment, Loyalty Points, Notification.

1. Define the happy path steps
2. Define compensating actions for each step
3. What happens if Payment fails after Room Reservation succeeds?
4. What happens if Room Reservation times out (no response)?
5. Choose orchestration or choreography and justify

### Exercise 3: Data Ownership Challenge

Your monolith has this SQL query used by the order history page:
```sql
SELECT o.id, o.status, o.created_at, c.name, c.email, 
       p.name as product_name, p.image_url, ol.quantity, ol.price
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_lines ol ON o.id = ol.order_id
JOIN products p ON ol.product_id = p.id
WHERE c.id = ?
ORDER BY o.created_at DESC
```

In microservices, this JOIN across 3 tables (3 services) is impossible. Design three different solutions and compare tradeoffs.

### Exercise 4: Monolith Decomposition Plan

Your company has a 500K LOC e-commerce monolith (Java/Spring, single PostgreSQL database, 8 developers). Create a decomposition plan:

1. First 3 services to extract and the order
2. Migration strategy for each (strangler fig approach)
3. How to handle the shared database during migration
4. Team structure changes needed
5. Infrastructure prerequisites
6. Timeline estimate and risk mitigation

---

## 11. Self-Check Questions

1. What is a bounded context and why is it important for microservices?
2. Explain the difference between entities and value objects. Give a Java example.
3. What is the aggregate rule "one transaction per aggregate" and why does it matter?
4. Why is the synchronous chain problem dangerous? How do you solve it?
5. Explain the Saga pattern. What is a compensating transaction?
6. What is contract testing and why is it important in microservices?
7. Name 3 microservices anti-patterns and how to avoid them.
8. When should you use gRPC over REST for inter-service communication?

---

## 12. Answers

### Answer 1
A bounded context is a boundary within which a domain model and its ubiquitous language are valid. "Product" in the Catalog context (SKU, description, images) is different from "Product" in the Shipping context (dimensions, weight). Important for microservices because each bounded context maps naturally to a service — it's the primary tool for finding correct service boundaries.

### Answer 2
**Entity** — has a unique identity; two entities with the same attributes are NOT the same if they have different IDs. Example: `Order` with `OrderId`. **Value Object** — defined entirely by its attributes; no identity concept. Two value objects with the same attributes ARE equal. Example: `Money(100, USD)` equals another `Money(100, USD)`. In Java, use `record` for value objects (immutable, auto-equals/hashCode).

### Answer 3
One transaction should only modify one aggregate. If you need to modify Order and Inventory in one operation, use an event — Order publishes `OrderPlaced`, Inventory listens and updates itself. This matters because: (a) aggregates may be in different databases/services, (b) keeping transactions small reduces contention and improves performance, (c) it forces you to think about consistency boundaries explicitly.

### Answer 4
If Service A synchronously calls B, which calls C, the overall availability is A × B × C. Three 99.9% services = 99.7% (3× more downtime). Solutions: (1) use async events instead of sync calls, (2) use circuit breakers with fallbacks, (3) cache results to reduce dependency, (4) design for eventual consistency.

### Answer 5
The Saga pattern replaces a single distributed transaction with a sequence of local transactions. Each step has a compensating transaction that undoes its effect if a later step fails. Example: If payment fails after inventory was reserved, the compensating action releases the inventory reservation. Sagas can be orchestrated (central coordinator) or choreographed (event-driven).

### Answer 6
Contract testing verifies that a provider's API meets the expectations of its consumers. Consumer writes a "pact" (expected request/response), provider verifies it passes. Important because: (a) it catches breaking API changes before deployment, (b) it's faster and more reliable than end-to-end tests, (c) it enables independent deployment — if contracts pass, the service is safe to deploy.

### Answer 7
(1) **Distributed Monolith** — services must deploy together. Fix: ensure independent deployability, remove synchronous chains. (2) **Shared Database** — multiple services access same tables. Fix: database per service, events for data sharing. (3) **Chatty Services** — too many fine-grained calls. Fix: coarser APIs, event-carried state transfer, aggregate data at the API gateway.

### Answer 8
Use gRPC when: (a) internal service-to-service communication with high throughput requirements, (b) you need strict API contracts enforced by Protocol Buffers, (c) you need streaming (bidirectional), (d) performance matters more than debuggability. Use REST when: public APIs, browser clients, external partners, or when tooling/debuggability is prioritized.

---

## References

- "Building Microservices" (2nd Ed.) — Sam Newman
- "Domain-Driven Design Distilled" — Vaughn Vernon
- "Implementing Domain-Driven Design" — Vaughn Vernon
- "Software Architecture: The Hard Parts" — Ford, Richards, Sadalage, Dehghani
- [microservices.io](https://microservices.io/) — Pattern catalog
- [Pact Contract Testing](https://pact.io/) — Consumer-driven contracts
- [Testcontainers](https://testcontainers.com/) — Integration testing

---

*Previous lesson: [02 — Architecture Styles](02-architecture-styles.md)*  
*Next lesson: [04 — Distributed Systems & Data](04-distributed-systems-data.md)*