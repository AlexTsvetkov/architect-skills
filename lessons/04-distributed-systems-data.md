# Lesson 04: Distributed Systems & Data Management

> **Prerequisites:** Lessons 01-03  
> **Time estimate:** 8-10 hours  
> **Parallel reading:** Finish "Designing Data-Intensive Applications" — Martin Kleppmann

---

## Table of Contents

1. [CAP Theorem — Practical Implications](#1-cap-theorem)
2. [Consistency Models](#2-consistency-models)
3. [Distributed Consensus](#3-distributed-consensus)
4. [Replication Strategies](#4-replication-strategies)
5. [Partitioning / Sharding](#5-partitioning--sharding)
6. [Distributed Caching](#6-distributed-caching)
7. [CQRS Implementation](#7-cqrs-implementation)
8. [Event Sourcing in Practice](#8-event-sourcing-in-practice)
9. [Change Data Capture](#9-change-data-capture)
10. [Polyglot Persistence](#10-polyglot-persistence)
11. [Exercises](#11-exercises)
12. [Self-Check Questions](#12-self-check-questions)
13. [Answers](#13-answers)

---

## 1. CAP Theorem

### The Theorem

In a distributed system experiencing a network partition, you must choose between **Consistency** and **Availability**.

```
    C — Consistency
   / \
  /   \      Pick two? No — pick one when P happens.
 /     \     P (partition) WILL happen. So: CP or AP.
A ───── P
Availability  Partition Tolerance
```

### What CAP Really Means

- **Consistency (C):** Every read receives the most recent write or an error
- **Availability (A):** Every request receives a response (not an error)
- **Partition Tolerance (P):** System continues despite network failures between nodes

**Key insight:** Network partitions ARE going to happen. So the real choice is:
- **CP:** During a partition, sacrifice availability (return errors) to maintain consistency
- **AP:** During a partition, sacrifice consistency (return stale data) to maintain availability

### Practical Examples

| System | Choice | Behavior During Partition |
|--------|--------|--------------------------|
| PostgreSQL (single node) | CA | No partitions possible (single node) |
| PostgreSQL (sync replicas) | CP | Writes blocked until replica confirms |
| Cassandra | AP | Returns data from available nodes (may be stale) |
| MongoDB (default) | CP | Writes only to primary; reads may fail if primary unreachable |
| DynamoDB | AP (tunable) | Returns data from available nodes |
| ZooKeeper | CP | Blocks writes without quorum |

### PACELC — Beyond CAP

```
If there is a Partition (P):
  Choose Availability (A) or Consistency (C)
Else (E) — when system is running normally:
  Choose Latency (L) or Consistency (C)

Examples:
- DynamoDB: PA/EL — available during partitions, low latency normally
- PostgreSQL sync: PC/EC — consistent always, higher latency
- Cassandra: PA/EL — available and fast, eventual consistency
- MongoDB: PA/EC — available during partitions but consistent normally
```

**This is more useful than CAP.** Most of the time your system isn't partitioned, and the L vs. C tradeoff matters more.

---

## 2. Consistency Models

### Strong Consistency

Every read sees the most recent write. As if there's a single copy of the data.

```
Client A writes: x = 5
Client B reads:  x → always returns 5 (or waits until available)

Implementation: synchronous replication, distributed locking
Cost: higher latency, lower availability
Use when: financial transactions, inventory counts, user authentication
```

### Eventual Consistency

If no new writes occur, all replicas will eventually converge to the same value.

```
Client A writes: x = 5 (to Node 1)
Client B reads from Node 2: x → might return 3 (old value)
... time passes, replication catches up ...
Client B reads from Node 2: x → returns 5

"Eventually" might be milliseconds or seconds.
```

### Causal Consistency

Operations that are causally related are seen in the same order by all nodes.

```
Alice posts: "Anyone want to grab lunch?"  (Event A)
Bob replies: "Sure, I'm in!"               (Event B, caused by A)

Causal consistency guarantees:
  Everyone sees A before B
  BUT unrelated posts may appear in different orders on different nodes
```

### Read-Your-Writes Consistency

A user always sees their own writes.

```
User updates profile name to "Alex"
User refreshes page → must see "Alex" (not old name)

Implementation: read from the leader for recently-written data
                or track write timestamps per user
```

### Choosing a Consistency Model

| Consistency | Latency | Availability | Use Case |
|-------------|---------|-------------|----------|
| Strong | High | Lower | Banking, stock trading |
| Causal | Medium | Medium | Social media, messaging |
| Read-your-writes | Low-Medium | High | User profiles, settings |
| Eventual | Low | Highest | Analytics, product catalog, recommendations |

---

## 3. Distributed Consensus

### Why Consensus Matters

Distributed systems need agreement on:
- Who is the leader? (leader election)
- What is the committed state? (replicated state machines)
- What is the configuration? (cluster membership)

### Raft Algorithm (Conceptual)

```
Node States: Follower → Candidate → Leader

1. LEADER ELECTION
   - Followers don't hear from leader → timeout
   - Become Candidate, request votes from others
   - Win majority → become Leader
   - Leader sends heartbeats to maintain authority

2. LOG REPLICATION
   - Client sends write to Leader
   - Leader appends to log, sends to Followers
   - Once majority acknowledge → committed
   - Leader notifies client: "success"

3. SAFETY
   - Only most up-to-date node can win election
   - Committed entries are never lost
```

### Where Consensus Is Used

| System | Uses Consensus For |
|--------|-------------------|
| etcd (Kubernetes) | Cluster state storage (Raft) |
| ZooKeeper | Configuration, leader election (ZAB) |
| Kafka | Partition leadership (KRaft, replacing ZooKeeper) |
| CockroachDB | Distributed SQL (Raft) |
| Consul | Service discovery, KV store (Raft) |

**As an architect, you don't implement consensus** — you choose systems that implement it correctly and understand their tradeoffs.

---

## 4. Replication Strategies

### Leader-Follower (Primary-Replica)

```
┌────────────┐    sync/async    ┌────────────┐
│   Leader   │ ───────────────→ │  Follower  │
│  (writes)  │                  │  (reads)   │
└────────────┘                  └────────────┘
      │                         ┌────────────┐
      └───────────────────────→ │  Follower  │
                                │  (reads)   │
                                └────────────┘

Writes: only to leader
Reads: from any follower (eventual consistency) or leader (strong)
Failover: promote a follower to leader if leader fails
```

**When to use:** Read-heavy workloads. Most relational databases.

### Multi-Leader (Multi-Primary)

```
┌────────────┐ ←──────────→ ┌────────────┐
│  Leader A  │              │  Leader B  │
│ (Region 1) │              │ (Region 2) │
└────────────┘              └────────────┘

Both accept writes. Replication in both directions.
Conflict resolution required when both modify same data.
```

**When to use:** Multi-region deployments (users write to nearest leader). Collaborative editing.

**Conflict resolution strategies:**
- Last-write-wins (LWW) — simple but data loss possible
- Merge — application-level merge logic
- CRDTs (Conflict-free Replicated Data Types) — data structures that auto-merge

### Leaderless (Dynamo-style)

```
Client writes to N nodes simultaneously.
Client reads from N nodes simultaneously.
Quorum: W + R > N ensures overlap.

Example (N=3, W=2, R=2):
Write to 2 of 3 nodes → success
Read from 2 of 3 nodes → at least one has latest value
```

**When to use:** Highest availability requirements. Cassandra, DynamoDB.

---

## 5. Partitioning / Sharding

### Why Partition?

When data is too large for a single node or queries are too heavy, split data across multiple nodes.

### Partitioning Strategies

#### Key-Range Partitioning

```
Partition A: customers A-H
Partition B: customers I-P
Partition C: customers Q-Z

Pros: Range queries efficient (get all customers M-N)
Cons: Hot spots if data isn't evenly distributed
```

#### Hash Partitioning

```
partition = hash(key) % num_partitions

Pros: Even distribution (no hot spots)
Cons: Range queries require querying all partitions
```

#### Consistent Hashing

```
      0 ────── Node A
     /          │
    /           │
 Node D      Node B
    \           │
     \          │
      ── Node C ─

Adding/removing a node only affects adjacent nodes.
Used by: Cassandra, DynamoDB, Redis Cluster
```

### Hot Spots and Solutions

**Problem:** One partition gets disproportionate traffic (e.g., celebrity user, popular product).

**Solutions:**
1. Add random suffix to hot key: `userId_0`, `userId_1` → spread across partitions
2. Application-level awareness: detect hot keys and split them
3. Caching layer in front of hot partitions

---

## 6. Distributed Caching

### Caching Strategies

#### Cache-Aside (Lazy Loading)

```
Read:
1. Check cache → hit? Return cached value
2. Cache miss → read from DB
3. Store in cache for next time

Write:
1. Write to DB
2. Invalidate cache (or let it expire via TTL)

Most common pattern. Application manages cache explicitly.
```

#### Read-Through

```
Read:
1. Always read from cache
2. Cache checks DB on miss (cache manages itself)

Write:
1. Write to DB
2. Invalidate cache

Cache library handles DB reads. Simpler application code.
```

#### Write-Through

```
Write:
1. Write to cache
2. Cache synchronously writes to DB

Read:
1. Always read from cache (always up-to-date)

Pros: Cache is always consistent with DB
Cons: Write latency (two writes per operation)
```

#### Write-Behind (Write-Back)

```
Write:
1. Write to cache
2. Cache asynchronously writes to DB (batched)

Pros: Very fast writes, batch DB updates
Cons: Risk of data loss if cache node fails before DB write
```

### Redis in Cloud-Native Architecture

```yaml
# Kubernetes Redis deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            memory: "256Mi"
          limits:
            memory: "512Mi"
```

**Common Redis use cases:**
- Session storage
- API response caching
- Rate limiting (sliding window counter)
- Distributed locks (Redlock)
- Pub/sub messaging
- Leaderboards (sorted sets)

### Cache Pitfalls

| Problem | Description | Solution |
|---------|-------------|----------|
| Cache stampede | Many requests hit DB simultaneously on cache miss | Locking (only one request populates cache) |
| Cache penetration | Requests for non-existent data always hit DB | Cache negative results (null with short TTL) |
| Stale data | Cache shows outdated information | TTL, event-driven invalidation |
| Memory pressure | Cache grows too large | Eviction policies (LRU, LFU), max memory limits |

---

## 7. CQRS Implementation

### Architecture

```
┌──────────────────────────────────────────────────────┐
│                     CQRS Architecture                 │
│                                                       │
│  COMMAND SIDE              │     QUERY SIDE            │
│                            │                           │
│  ┌─────────────┐          │     ┌──────────────┐      │
│  │ Command     │          │     │ Query        │      │
│  │ Handler     │          │     │ Handler      │      │
│  └──────┬──────┘          │     └──────┬───────┘      │
│         │                  │            │              │
│  ┌──────┴──────┐          │     ┌──────┴───────┐      │
│  │ Domain      │          │     │ Read Model   │      │
│  │ Model       │          │     │ (Projections)│      │
│  └──────┬──────┘          │     └──────┬───────┘      │
│         │                  │            │              │
│  ┌──────┴──────┐   Events │     ┌──────┴───────┐      │
│  │ Write DB    │ ─────────┼──→  │ Read DB      │      │
│  │ (PostgreSQL)│          │     │ (Elasticsearch│      │
│  └─────────────┘          │     │  / Redis /   │      │
│                            │     │  DynamoDB)   │      │
└──────────────────────────────────────────────────────┘
```

### Spring Boot CQRS Example

```java
// COMMAND SIDE
@Service
public class OrderCommandService {
    @Autowired private OrderRepository orderRepo;
    @Autowired private ApplicationEventPublisher events;

    @Transactional
    public OrderId createOrder(CreateOrderCommand cmd) {
        Order order = Order.create(cmd.getCustomerId(), cmd.getItems());
        orderRepo.save(order);
        events.publishEvent(new OrderCreated(order.getId(), order.getItems()));
        return order.getId();
    }
}

// QUERY SIDE
@Service
public class OrderQueryService {
    @Autowired private OrderReadRepository readRepo; // Elasticsearch

    public OrderView getOrder(String orderId) {
        return readRepo.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
    }

    public List<OrderView> searchOrders(OrderSearchCriteria criteria) {
        return readRepo.search(criteria);
    }
}

// PROJECTION (Event Handler that updates read model)
@Component
public class OrderProjection {
    @Autowired private OrderReadRepository readRepo;

    @EventListener
    public void on(OrderCreated event) {
        OrderView view = new OrderView(
            event.getOrderId(),
            event.getItems(),
            "CREATED"
        );
        readRepo.save(view);
    }

    @EventListener
    public void on(OrderShipped event) {
        OrderView view = readRepo.findById(event.getOrderId());
        view.setStatus("SHIPPED");
        view.setTrackingNumber(event.getTracking());
        readRepo.save(view);
    }
}
```

---

## 8. Event Sourcing in Practice

### Event Store Design

```
┌────────────────────────────────────────────────────┐
│                    EVENT STORE                       │
├──────────┬───────────┬───────────┬────────┬────────┤
│ event_id │ aggregate │ aggregate │ event  │ data   │
│          │ _type     │ _id       │ _type  │ (JSON) │
├──────────┼───────────┼───────────┼────────┼────────┤
│ 1        │ Order     │ ORD-123   │ Created│ {...}  │
│ 2        │ Order     │ ORD-123   │ ItemAdd│ {...}  │
│ 3        │ Order     │ ORD-123   │ Submit │ {...}  │
│ 4        │ Order     │ ORD-456   │ Created│ {...}  │
│ 5        │ Order     │ ORD-123   │ Paid   │ {...}  │
└──────────┴───────────┴───────────┴────────┴────────┘

To get current state of ORD-123:
  Load events 1, 2, 3, 5
  Replay them to reconstruct state
```

### Snapshots

For aggregates with many events, replaying everything is slow. Use snapshots:

```
Events:    [1] [2] [3] [4] [5] ... [500] [501] [502]
                                    ↑
                              Snapshot at event 500
                              (full state at this point)

To load: Read snapshot + replay events 501, 502
Much faster than replaying all 502 events.
```

### Event Schema Evolution

```
// Version 1 of the event
{ "type": "OrderCreated", "orderId": "123", "customer": "Alice" }

// Version 2 — added items field
{ "type": "OrderCreated", "orderId": "123", "customerId": "C-456", 
  "items": [...] }

Strategies:
1. Upcasting — transform old events to new format on read
2. Versioned events — OrderCreatedV1, OrderCreatedV2
3. Lazy migration — only transform when replaying
```

---

## 9. Change Data Capture (CDC)

### What is CDC?

Capture changes from a database and publish them as events.

```
┌─────────────┐    INSERT/UPDATE/DELETE    ┌──────────────┐
│  Application │ ─────────────────────────→│   Database   │
└─────────────┘                            └──────┬───────┘
                                                  │ CDC
                                                  │ (reads DB log)
                                           ┌──────┴───────┐
                                           │  Debezium /  │
                                           │  CDC Tool    │
                                           └──────┬───────┘
                                                  │
                                           ┌──────┴───────┐
                                           │    Kafka     │
                                           └──────┬───────┘
                                                  │
                              ┌────────────┬──────┴──────┐
                              │            │             │
                         Search Index  Analytics    Other Services
                         (Elasticsearch) (Data Lake)
```

### When to Use CDC

1. **Sync data between services** without changing application code
2. **Build read models** for CQRS from legacy databases
3. **Feed data pipelines** and analytics
4. **Migrate from monolith** — capture changes from the monolith DB, feed new microservices
5. **Outbox pattern** — capture outbox table changes for reliable event publishing

### Tools

| Tool | Database Support | Target |
|------|-----------------|--------|
| Debezium | PostgreSQL, MySQL, MongoDB, Oracle, SQL Server | Kafka |
| AWS DMS | Most databases | Various AWS services |
| Azure CDC | SQL Server, Cosmos DB | Event Hubs |

---

## 10. Polyglot Persistence

### Choose the Right Database for Each Service

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Order Service │  │ Product Svc  │  │ Session Svc  │
│               │  │              │  │              │
│  PostgreSQL   │  │ Elasticsearch│  │    Redis     │
│  (ACID,       │  │ (full-text   │  │  (fast K/V,  │
│   relations)  │  │  search,     │  │   TTL-based) │
│               │  │  faceted)    │  │              │
└──────────────┘  └──────────────┘  └──────────────┘

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Analytics Svc │  │  Graph Svc   │  │  Event Store │
│               │  │              │  │              │
│  ClickHouse   │  │    Neo4j     │  │ EventStoreDB │
│  (columnar,   │  │ (relations,  │  │ (event       │
│   analytics)  │  │  traversals) │  │  sourcing)   │
└──────────────┘  └──────────────┘  └──────────────┘
```

### Database Decision Guide

| Need | Best Fit | Examples |
|------|----------|---------|
| ACID transactions, relations | Relational | PostgreSQL, MySQL |
| Full-text search, faceted | Search engine | Elasticsearch, Solr |
| High-speed K/V, caching | Key-value | Redis, Memcached |
| Document/JSON, flexible schema | Document | MongoDB, DynamoDB |
| Relationships, graph queries | Graph | Neo4j, Amazon Neptune |
| Time-series data | Time-series | InfluxDB, TimescaleDB |
| Analytics, OLAP | Columnar | ClickHouse, BigQuery |
| Event sourcing | Event store | EventStoreDB, Kafka |

---

## 11. Exercises

### Exercise 1: Consistency Model Design

Design the consistency requirements for an e-commerce platform. For each feature, specify:
- The consistency model (strong, eventual, causal, read-your-writes)
- The justification
- The implementation approach

Features: (a) Inventory count, (b) Product catalog, (c) Shopping cart, (d) Order history, (e) Payment processing, (f) Product reviews, (g) Price display

### Exercise 2: Sharding Strategy

You have a table with 500M user records growing by 1M/day. Design a sharding strategy:
1. Choose a partition key (and justify why)
2. Choose a partitioning method (range vs. hash vs. consistent hashing)
3. Handle cross-shard queries (e.g., "find all users in Germany")
4. Plan for resharding when you add nodes

### Exercise 3: Caching Architecture

Design a caching architecture for a product catalog service:
- 50M products
- 10M reads/day, 10K writes/day
- Average product page loads 5 related products
- Products have images, descriptions, prices, reviews

Specify: cache strategy, cache granularity, TTL policy, invalidation approach, cache topology.

### Exercise 4: CQRS for E-Commerce

Implement a CQRS architecture for an order management system:
1. Define the command model (write side)
2. Define at least 3 different read models (query side)
3. Design the synchronization mechanism between write and read
4. Handle eventual consistency (what happens when a user creates an order and immediately views it?)

---

## 12. Self-Check Questions

1. Explain the CAP theorem. Why is "CA" not a real option in a distributed system?

2. What is the difference between eventual consistency and strong consistency? Give a real-world example of each.

3. When would you choose leader-follower vs. leaderless replication?

4. What is the difference between hash partitioning and range partitioning? When would you choose each?

5. Explain the cache-aside pattern. What is the "cache stampede" problem and how do you solve it?

6. What is Change Data Capture? Give two practical use cases.

7. A team wants to use event sourcing for a CRUD settings page. What would you advise?

8. What is PACELC and why is it more useful than CAP?

---

## 13. Answers

### Answer 1
The CAP theorem states that during a network partition, a distributed system must choose between consistency (every read gets the latest write) and availability (every request gets a response). "CA" is not practical because **network partitions are inevitable** in distributed systems. A single-node database is "CA" but it's not distributed. The moment you distribute data across nodes, partitions can occur, and you must handle them — either by sacrificing availability (CP) or consistency (AP).

### Answer 2
**Strong consistency:** After a write completes, all subsequent reads see that write. Example: Bank transfer — after transferring $100, both sender and receiver immediately see updated balances. Implementation: synchronous replication, distributed locks.

**Eventual consistency:** After a write, replicas will converge to the same state eventually, but reads may return stale data temporarily. Example: Social media "like" count — if you like a post, other users might see the old count for a few seconds. Implementation: asynchronous replication, conflict resolution.

### Answer 3
**Leader-follower:** When you need strong consistency for writes, have a clear write path, and read-heavy workloads where reads can tolerate slight staleness. Most relational databases use this. Simpler to reason about.

**Leaderless:** When you need highest availability (no single point of failure), can tolerate eventual consistency, and need multi-region writes. Used by Cassandra, DynamoDB. More complex conflict resolution but no leader failover needed.

### Answer 4
**Hash partitioning:** `partition = hash(key) % N`. Even distribution, no hot spots. But range queries require scatter-gather across all partitions. Choose when: access pattern is primarily key-based lookups.

**Range partitioning:** Split by key ranges (A-H, I-P, Q-Z). Efficient range queries within a partition. But risk of hot spots if data distribution is skewed. Choose when: range queries are common (e.g., time-series data partitioned by date range).

### Answer 5
**Cache-aside:** Application checks cache first. On miss, reads from DB, stores in cache. On write, writes to DB and invalidates cache. Application controls caching logic.

**Cache stampede:** When a popular cache key expires, hundreds of concurrent requests all miss the cache simultaneously and all hit the database. This can overwhelm the DB.

**Solutions:** (1) Locking — only one request fetches from DB, others wait. (2) Probabilistic early expiration — randomly refresh before TTL. (3) Background refresh — a background job refreshes popular keys before expiration.

### Answer 6
**CDC** captures row-level changes (INSERT, UPDATE, DELETE) from a database transaction log and publishes them as events.

**Use case 1:** Building search indexes. A product service writes to PostgreSQL. CDC captures changes and streams them to Elasticsearch for full-text search. No application code changes needed.

**Use case 2:** Monolith to microservices migration. The monolith writes to its database. CDC captures changes and feeds them to new microservices, allowing gradual migration without modifying the monolith.

### Answer 7
Event sourcing for a CRUD settings page is **overkill**. Settings pages have simple read/write patterns, don't need audit trails (or a simple `updated_at` column suffices), and don't benefit from event replay. The complexity of event schema evolution, projections, and eventual consistency is not justified. Use a simple relational table with standard CRUD operations. Reserve event sourcing for domains with complex state transitions, audit requirements, or temporal query needs.

### Answer 8
**PACELC** extends CAP: if there is a **P**artition, choose **A** or **C**; **E**lse (normal operation), choose **L**atency or **C**onsistency. It's more useful because most of the time your system isn't partitioned. The daily tradeoff is latency vs. consistency, not availability vs. consistency. For example, DynamoDB is PA/EL — available during partitions and optimizes for low latency normally. PostgreSQL with synchronous replication is PC/EC — always consistent but higher latency.

---

*Previous: [03 - Microservices Deep Dive](03-microservices-deep-dive.md)*  
*Next: [05 - Containers & Orchestration](05-containers-orchestration.md)*