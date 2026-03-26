# Prompt: Lesson 12 — Scalability & Performance

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 12: Scalability & Performance**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot. Builds on System Design Interview (Alex Xu) concepts. Reference: Microsoft Performance Best Practices.

### Topic Coverage

1. **Horizontal vs Vertical Scaling**
   - Definitions with diagrams
   - When to use each (table with scenarios)
   - Cloud-native default: horizontal

2. **Auto-Scaling**
   - Kubernetes HPA (CPU/memory-based, YAML example)
   - Kubernetes VPA (right-sizing recommendations)
   - KEDA (event-driven autoscaling: Kafka lag, queue depth)
   - Cluster autoscaler
   - Scaling decision flowchart

3. **Load Balancing**
   - L4 vs L7 comparison table
   - Algorithms: Round Robin, Weighted Round Robin, Least Connections, IP Hash, Consistent Hashing
   - Kubernetes Service load balancing
   - Ingress controller load balancing

4. **CDN and Edge Computing**
   - CDN architecture diagram
   - Cache-Control headers
   - Edge computing use cases
   - Static vs dynamic content caching

5. **Caching Strategies** (4 patterns with diagrams)
   - Cache-Aside
   - Read-Through
   - Write-Through
   - Write-Behind
   - Cache invalidation strategies
   - Redis cluster architecture

6. **Database Scaling**
   - Read replicas (diagram, Spring Boot configuration)
   - Sharding strategies
   - Connection pooling (HikariCP configuration)
   - Query optimization and indexing

7. **Asynchronous Processing**
   - Work queues pattern
   - CompletableFuture for async processing
   - Virtual threads (Java 21) for I/O-bound work
   - Spring @Async with thread pool configuration

8. **Performance Testing**
   - Load testing vs stress testing vs soak testing
   - Tools: JMeter, Gatling, k6
   - Performance testing in CI/CD
   - Performance budgets

9. **Capacity Planning**
   - Little's Law: L = λW
   - Back-of-the-envelope calculations
   - Growth projection

10. **Back Pressure and Flow Control**
    - The problem: fast producer, slow consumer
    - Reactive Streams back pressure
    - Rate limiting as back pressure
    - Queue depth monitoring + autoscaling

### Code Examples Required
1. HPA YAML configuration (CPU + custom metrics)
2. KEDA ScaledObject YAML (Kafka consumer lag trigger)
3. Spring Boot read replica routing (@Transactional(readOnly=true))
4. HikariCP connection pool configuration
5. Spring @Async with thread pool
6. Java 21 virtual threads example
7. Back pressure with Project Reactor

### Diagrams Required (minimum 8)
Horizontal vs vertical scaling, Auto-scaling decision flow, L4 vs L7 load balancing, CDN architecture, Cache-aside pattern, Read replica routing, Work queue pattern, Back pressure flow

### Real-World Case Study
Design system to handle 10x traffic spike during Black Friday for e-commerce: auto-scaling strategy, caching, CDN, database scaling, queue-based processing, capacity planning.

### Exercises (4)
1. Auto-scaling design for e-commerce (HPA, VPA, KEDA, cluster autoscaler)
2. Caching strategy for product catalog (pattern selection, TTL, invalidation, topology)
3. Database scaling for order service (read replicas, connection pooling, query optimization)
4. Black Friday preparation (capacity planning, performance testing, gradual rollout)

### Self-Check Questions (10)
Cover: horizontal vs vertical, HPA vs VPA vs KEDA, L4 vs L7, caching strategies, read replicas, connection pooling, performance testing types, capacity planning (Little's Law), back pressure, CDN cache invalidation.

### References
Include: "System Design Interview" (Alex Xu), Google SRE Book, Microsoft Performance Best Practices, Spring Boot performance tuning docs, Kubernetes autoscaling docs.