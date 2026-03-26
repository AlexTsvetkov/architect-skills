# Prompt: Lesson 02 — Architecture Styles & Patterns

```markdown
You are an expert cloud-native software architect and technical educator. Your task is to create **Lesson 02: Architecture Styles & Patterns** for a professional-level course on cloud-native architecture.

### Context

Course for experienced Java developers (5+ years) transitioning to architect roles. Deep, technical, practical. Primary stack: Java/Spring Boot. Reference: Microsoft Azure Architecture Styles Guide (https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/).

### Topic: Architecture Styles & Patterns

Cover each architecture style with: overview, diagram, strengths/weaknesses table, when to use/avoid, and enterprise Java context.

1. **How to Think About Architecture Styles**
   - Architecture characteristics (quality attributes): scalability, elasticity, availability, reliability, performance, deployability, testability, modularity, fault tolerance, cost, simplicity, evolvability
   - The Laws of Software Architecture (Richards & Ford): "Everything is a tradeoff"
   - Architecture styles overview map (monolithic → distributed spectrum)

2. **N-Tier / Layered Architecture**
   - Classic 3-tier with Spring Boot project structure
   - Architecture sinkhole anti-pattern with code example
   - Enterprise monolith context

3. **Microservices Architecture**
   - Key characteristics (6 principles)
   - When to use vs avoid (table)
   - The "microservices premium" concept
   - Microsoft reference architecture diagram

4. **Event-Driven Architecture**
   - Two models: Event Notification (Pub/Sub) and Event Streaming
   - Comparison table (storage, replay, ordering, coupling)
   - Three event types: notification, event-carried state transfer, event sourcing

5. **CQRS & Event Sourcing**
   - CQRS architecture with separate read/write models
   - When to use CQRS (table)
   - Event Sourcing: sequence of events vs current state
   - Benefits and risks (5 each)

6. **Web-Queue-Worker**
   - Architecture diagram
   - E-commerce order processing example
   - Microsoft reference link

7. **Big Data Architectures**
   - Lambda architecture (batch + speed layers)
   - Kappa architecture (streaming only)
   - Comparison table
   - DDIA connection

8. **Choreography vs. Orchestration**
   - Visual comparison of both approaches
   - Detailed comparison table (7 aspects)
   - Saga pattern introduction with compensating actions

9. **Space-Based Architecture**
   - Architecture diagram with processing units, data grid, data pump
   - Use cases (ticketing, auctions, trading)

10. **Micro-Frontends**
    - Vertical team ownership diagram
    - Integration approaches table (5 approaches)

11. **Modular Monolith**
    - Architecture diagram (single deployable with module boundaries)
    - Spring Boot multi-module implementation
    - ArchUnit boundary enforcement code example
    - Migration path: Monolith → Modular Monolith → Microservices

12. **Choosing the Right Style**
    - Decision framework/flowchart
    - Architecture style comparison matrix (7 styles × 5 attributes)

### Code Examples Required

1. **Architecture sinkhole anti-pattern** — Controller→Service→Repository passthrough
2. **Modular monolith project structure** — Multi-module Maven project with api/internal separation
3. **ArchUnit boundary tests** — Enforcing module boundaries programmatically

### Diagrams Required (minimum 8)

1. Architecture styles spectrum (monolithic → distributed)
2. Layered architecture
3. Microservices with API gateway
4. Event notification (pub/sub) flow
5. Event streaming flow
6. CQRS architecture
7. Orchestration vs choreography comparison
8. Modular monolith structure
9. Decision framework flowchart

### Exercises (4)

1. **Architecture Style Mapping** — Match 6 systems to appropriate styles with justification
2. **Architecture Tradeoff Analysis** — Write proposal for monolith-to-microservices migration
3. **CQRS Decision** — Design CQRS and non-CQRS approaches for product catalog, compare
4. **Event-Driven Design** — Design event-driven order processing with events, services, failure handling

### Self-Check Questions (10)

Cover: quality attribute conflicts, sinkhole anti-pattern, modular monolith criteria, event types, event sourcing appropriate use, choreography vs orchestration, CQRS + event sourcing relationship, Lambda vs Kappa, Web-Queue-Worker, smart endpoints/dumb pipes.

### References

Include: Microsoft Architecture Styles Guide, "Fundamentals of Software Architecture", Martin Fowler's architecture articles, microservices.io, Azure Architecture Guide.