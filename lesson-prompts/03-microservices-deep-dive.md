# Prompt: Lesson 03 — Microservices Architecture Deep Dive

```markdown
You are an expert cloud-native software architect and technical educator. Your task is to create **Lesson 03: Microservices Architecture — Deep Dive** for a professional-level course on cloud-native architecture.

### Context

Course for experienced Java developers transitioning to architect roles. Deep, technical, practical. Primary stack: Java/Spring Boot. This is the longest and most detailed lesson in the course.

### Topic: Microservices Architecture — Deep Dive

1. **Domain-Driven Design for Microservices**
   - Strategic DDD: Bounded Contexts (with e-commerce example showing Product in 3 contexts)
   - Ubiquitous Language (table showing same term in different contexts)
   - Context Map with relationship types (U/D, ACL, Partnership, Shared Kernel, Conformist, Open Host Service)
   - Tactical DDD: Aggregates with Java code example (Order aggregate root with OrderLine, Money value objects)
   - Key rule: aggregates reference other aggregates by ID only
   - DDD to Microservices mapping table

2. **Decomposition Strategies**
   - Strategy 1: By Business Capability (tree structure)
   - Strategy 2: By Subdomain — Core/Supporting/Generic domains
   - Strategy 3: Strangler Fig Pattern (3-phase migration)
   - How to identify service boundaries (5 questions checklist)

3. **Data Ownership**
   - Database-per-service pattern (visual: shared DB anti-pattern vs per-service DBs)
   - Handling cross-service queries: API Composition, CQRS with read model, data replication via events
   - Tradeoff: denormalization + eventual consistency vs runtime coupling

4. **Inter-Service Communication**
   - Synchronous: REST vs gRPC comparison table (6 aspects)
   - Asynchronous: Commands vs Events comparison table
   - The synchronous chain problem (availability math: 99.9%^5 = 99.5%)

5. **API Gateway Pattern**
   - Responsibilities diagram
   - BFF (Backend for Frontend) pattern
   - Technology comparison table (Kong, AWS API GW, Azure APIM, Spring Cloud Gateway, Envoy)

6. **Service Discovery**
   - Client-side vs server-side discovery
   - Kubernetes built-in service discovery (YAML example)
   - Why Eureka is unnecessary in Kubernetes

7. **Distributed Transactions — Saga Pattern**
   - Why ACID doesn't work across services
   - Orchestration-based saga (step-by-step with compensation)
   - Choreography-based saga (event flow)
   - When to choose each (comparison table)

8. **Testing Microservices**
   - Testing pyramid for microservices (E2E → Contract → Integration → Unit)
   - Contract testing with Pact (concept + consumer test code)
   - Integration testing with Testcontainers (code example)
   - Chaos engineering principles and tools

9. **Anti-Patterns (5)**
   - Distributed Monolith (detection signs)
   - Nano-Services (too granular)
   - Shared Database
   - No API Versioning
   - Chatty Communication

### Code Examples Required

1. **DDD Aggregate Root** — Order class with OrderLine, Money, business rules, domain events
2. **Contract Testing with Pact** — Consumer-side Pact test for inventory service
3. **Integration Testing with Testcontainers** — Spring Boot test with PostgreSQL + Kafka containers
4. **Kubernetes Service Discovery** — YAML for Service + how other services call it

### Diagrams Required (minimum 7)

1. Bounded contexts for e-commerce (Catalog, Order, Shipping)
2. Database-per-service vs shared database
3. API Gateway with services
4. BFF pattern
5. Orchestration-based saga flow
6. Choreography-based saga flow
7. Testing pyramid for microservices
8. Strangler fig migration phases

### Exercises (4)

1. **Domain Decomposition** — Identify 6 bounded contexts, draw context map, define aggregates and events
2. **Saga Design** — Design saga for hotel booking system (both approaches, failure handling)
3. **Data Ownership Challenge** — Break monolith JOIN queries into microservices data ownership
4. **Monolith Decomposition** — Full decomposition plan for 500K LOC e-commerce monolith

### Self-Check Questions (10)

Cover: bounded contexts, aggregates/entities/value objects, database-per-service tradeoffs, gRPC vs REST, distributed monolith, saga pattern, contract testing, premature decomposition advice, strangler fig, BFF pattern.

### References

Include: "Building Microservices" (Sam Newman), "DDD Distilled" (Vaughn Vernon), microservices.io, Microsoft Microservices Architecture Guide, "Software Architecture: The Hard Parts".