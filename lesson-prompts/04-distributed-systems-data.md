# Prompt: Lesson 04 — Distributed Systems & Data Management

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 04: Distributed Systems & Data Management**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot stack. Students have partially read DDIA (Kleppmann). This lesson fills gaps and extends to practical implementation.

### Topic Coverage

1. **CAP Theorem — Practical Implications**
   - What C, A, P really mean (not the oversimplified version)
   - Why "pick two" is misleading — real choice is CP or AP during partition
   - Practical examples table: PostgreSQL, Cassandra, MongoDB, DynamoDB, ZooKeeper
   - PACELC theorem — more useful than CAP (with examples matrix)

2. **Consistency Models** (4 models with code/pseudocode examples)
   - Strong consistency (implementation, cost, use cases)
   - Eventual consistency (convergence timeline, examples)
   - Causal consistency (social media example)
   - Read-your-writes consistency (implementation strategies)
   - Choosing a consistency model (decision table)

3. **Distributed Consensus**
   - Why consensus matters (leader election, committed state, configuration)
   - Raft algorithm conceptual walkthrough (leader election, log replication, safety)
   - Where consensus is used (table: etcd, ZooKeeper, Kafka KRaft, CockroachDB, Consul)
   - Architect's perspective: choose systems with correct consensus, don't implement

4. **Replication Strategies** (3 strategies with diagrams)
   - Leader-Follower: diagram, when to use, failover
   - Multi-Leader: diagram, conflict resolution (LWW, merge, CRDTs)
   - Leaderless (Dynamo-style): quorum formula, diagram

5. **Partitioning / Sharding**
   - Key-range partitioning (pros/cons)
   - Hash partitioning (pros/cons)
   - Consistent hashing (ring diagram)
   - Hot spots and 3 solutions

6. **Distributed Caching** (4 strategies with diagrams)
   - Cache-Aside (lazy loading)
   - Read-Through
   - Write-Through
   - Write-Behind (write-back)
   - Redis in cloud-native architecture (K8s deployment YAML, use cases)
   - Cache pitfalls table (stampede, penetration, stale data, memory pressure)

7. **CQRS Implementation**
   - Full architecture diagram
   - Spring Boot implementation: CommandService, QueryService, Projection (event handler)

8. **Event Sourcing in Practice**
   - Event store schema design
   - Snapshots for performance
   - Event schema evolution strategies (upcasting, versioned events, lazy migration)

9. **Change Data Capture (CDC)**
   - Architecture diagram (App → DB → Debezium → Kafka → consumers)
   - 5 use cases
   - Tools comparison table (Debezium, AWS DMS, Azure CDC)

10. **Polyglot Persistence**
    - Database decision guide table (8 needs → best fit → examples)
    - Service-to-database mapping diagram

### Code Examples Required

1. **CQRS with Spring Boot** — Complete CommandService, QueryService, and Projection with event handling
2. **Redis Kubernetes deployment** — YAML manifest
3. **Cache-aside pattern** — Java implementation with Redis (show cache miss, hit, invalidation)

### Diagrams Required (minimum 8)
CAP triangle, Leader-Follower replication, Multi-Leader, Leaderless quorum, Consistent hashing ring, CQRS architecture, Event store schema, CDC pipeline, Polyglot persistence map

### Exercises (4)
1. Consistency model design for e-commerce features
2. Sharding strategy for 500M user records
3. Caching architecture for product catalog
4. CQRS for order management system

### Self-Check Questions (8)
Cover: CAP theorem, consistency models, replication strategies, partitioning, cache-aside + stampede, CDC, event sourcing appropriateness, PACELC.