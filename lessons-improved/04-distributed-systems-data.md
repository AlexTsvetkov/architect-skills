# Lesson 04: Distributed Systems & Data Management

> **Prerequisites:** Lessons 01-03  
> **Time estimate:** 8-10 hours  
> **Parallel reading:** "Designing Data-Intensive Applications" — Martin Kleppmann (Ch 5-9)

---

## Table of Contents

1. [CAP Theorem — Practical Implications](#1-cap-theorem--practical-implications)
2. [Consistency Models](#2-consistency-models)
3. [Distributed Consensus](#3-distributed-consensus)
4. [Replication Strategies](#4-replication-strategies)
5. [Partitioning / Sharding](#5-partitioning--sharding)
6. [Distributed Caching](#6-distributed-caching)
7. [CQRS Implementation](#7-cqrs-implementation)
8. [Event Sourcing in Practice](#8-event-sourcing-in-practice)
9. [Change Data Capture (CDC)](#9-change-data-capture-cdc)
10. [Polyglot Persistence](#10-polyglot-persistence)
11. [Exercises](#11-exercises)
12. [Self-Check Questions](#12-self-check-questions)
13. [Answers](#13-answers)

---

## 1. CAP Theorem — Practical Implications

### What C, A, P Really Mean

| Letter | Formal Definition | NOT What Most People Think |
|--------|------------------|---------------------------|
| **C** — Consistency | Every read receives the most recent write or an error (linearizability) | NOT "all nodes have the same data eventually" |
| **A** — Availability | Every request to a non-failing node receives a response (no errors, no timeouts) | NOT "the system is up" |
| **P** — Partition Tolerance | The system continues to operate despite network partitions between nodes | NOT optional — network partitions WILL happen |

### Why "Pick Two" Is Misleading

```
The CAP theorem says: During a network partition, you must choose
between Consistency and Availability.

NOT "pick two out of three" — partitions are not optional!

┌──────────────────────────────────────────────────────┐
│           NORMAL OPERATION (no partition)              │
│   You get BOTH consistency AND availability            │
│   CAP theorem doesn't apply!                          │
├──────────────────────────────────────────────────────┤
│           DURING A PARTITION                          │
│                                                       │
│   CP: Refuse requests that might return stale data    │
│       → Some requests fail (sacrifice availability)   │
│       Example: Node can't reach leader, returns error │
│                                                       │
│   AP: Continue serving requests with potentially      │
│       stale data (sacrifice consistency)              │
│       Example: Both partitions accept writes,         │
│                diverging state                        │
└──────────────────────────────────────────────────────┘
```

### Real Database CAP Classification

| Database | Partition Behavior | Category | Notes |
|----------|-------------------|----------|-------|
| **PostgreSQL** (single node) | No partitions — single node | CA* | *Not distributed; add Patroni for HA |
| **PostgreSQL** (streaming replication) | Followers may lag; during partition, promote follower → split brain possible | CP | Synchronous replication = CP |
| **CockroachDB** | Raft consensus; minority partition becomes unavailable | CP | Linearizable reads |
| **Cassandra** | Tunable: QUORUM = CP-ish, ONE = AP | AP (tunable) | Conflict resolution via LWW |
| **DynamoDB** | Eventually consistent reads default; optional strong reads | AP (tunable) | Strong consistency for specific reads |
| **MongoDB** | Primary election during partition; writes to primary only | CP | Reads from secondaries are AP |
| **ZooKeeper** | Minority partition can't serve writes | CP | Designed for coordination, not data storage |
| **Redis Cluster** | Async replication; during partition, writes may be lost | AP | Not suitable as primary data store |

### PACELC Theorem — More Useful Than CAP

> If there is a **P**artition, choose **A**vailability or **C**onsistency;  
> **E**lse (normal operation), choose **L**atency or **C**onsistency.

| System | P→A or C | E→L or C | Explanation |
|--------|----------|----------|-------------|
| **PostgreSQL** | PC | EC | Always consistent, higher latency for sync replication |
| **CockroachDB** | PC | EC | Consistent at cost of consensus latency |
| **Cassandra** | PA | EL | Available during partition, low latency in normal operation |
| **DynamoDB** (default) | PA | EL | Eventual consistency, low latency |
| **DynamoDB** (strong) | PC | EC | Strong consistency reads have higher latency |
| **MongoDB** | PC | EC | Consistent writes to primary |
| **Kafka** | PA | EL | Async replication, consumer lag acceptable |

---

## 2. Consistency Models

### Four Consistency Models Explained

```
STRONG CONSISTENCY (Linearizability)
────────────────────────────────────
Client A:  WRITE x=5  ──────────────────────────→
Client B:            READ x → waits... → returns 5 ✅
                     (guaranteed to see the latest write)

Implementation: Consensus protocol (Raft/Paxos), synchronous replication
Cost: Higher latency (must wait for majority acknowledgment)
Use when: Financial transactions, inventory counts, leader election


EVENTUAL CONSISTENCY
────────────────────
Client A:  WRITE x=5  ─────────────────────────→
Client B:  READ x → returns 3 (stale!)          READ x → returns 5 ✅
           │                                     │
           └── Convergence window (ms to sec) ───┘

Implementation: Async replication, gossip protocol
Cost: Stale reads possible (but low latency)
Use when: Social media likes, view counters, DNS


CAUSAL CONSISTENCY
──────────────────
Client A:  POST "Hello"  ──→  REPLY "Thanks" ──→
Client B:  READ → sees "Hello" then "Thanks" ✅  (preserves causal order)
Client B:  READ → sees "Thanks" without "Hello" ❌ (violates causality)

Implementation: Vector clocks, causal dependency tracking
Cost: Moderate complexity, moderate latency
Use when: Social media feeds (comments after posts), chat applications


READ-YOUR-WRITES CONSISTENCY
─────────────────────────────
Client A:  WRITE x=5 → READ x → returns 5 ✅ (always sees own writes)
Client B:  READ x → may return 3 (stale) → eventually returns 5

Implementation strategies:
  1. Read from leader after own writes (for a time window)
  2. Track write timestamp, read from replica only if replica >= timestamp
  3. Sticky sessions — always route to same replica

Use when: User profile updates ("I changed my name, I should see it")
```

### Consistency Model Decision Table

| Requirement | Model | Example |
|------------|-------|---------|
| Bank account balance must always be accurate | Strong | PostgreSQL with sync replication |
| Product review count can be slightly behind | Eventual | Cassandra, DynamoDB |
| Chat messages must appear in correct order | Causal | Custom with vector clocks |
| User sees their own profile changes immediately | Read-your-writes | Sticky sessions + leader reads |
| Shopping cart across multiple devices | Session + Eventual | Redis with conflict merge |

---

## 3. Distributed Consensus

### Why Consensus Matters

Distributed systems need agreement on: leader election, transaction commit/abort, configuration values, membership changes. Without consensus, split-brain and data loss.

### Raft Algorithm — Conceptual Walkthrough

```
RAFT — Three roles: Leader, Follower, Candidate

1. LEADER ELECTION
   ┌─────────┐    ┌─────────┐    ┌─────────┐
   │Follower │    │Follower │    │Follower │
   │ Node A  │    │ Node B  │    │ Node C  │
   └────┬────┘    └────┬────┘    └────┬────┘
        │              │              │
   No heartbeat from leader...        │
   Election timeout!                  │
        │              │              │
   ┌────┴────┐         │              │
   │Candidate│──Vote?──→              │
   │ Node A  │──Vote?────────────────→│
   └────┬────┘         │              │
        │         Vote YES        Vote YES
        │←─────────────│←─────────────│
        │              │              │
   ┌────┴────┐         │              │
   │ LEADER  │  Majority (2/3) votes  │
   │ Node A  │  → becomes Leader      │
   └─────────┘                        │

2. LOG REPLICATION
   Leader:   [Entry1] [Entry2] [Entry3]
              │         │        │
              ▼         ▼        ▼
   Follower: [Entry1] [Entry2] [Entry3]  ← replicated
   Follower: [Entry1] [Entry2]           ← catching up

   Entry is "committed" when majority of nodes have it.

3. SAFETY
   - Only one leader per term
   - Leader never overwrites its own log
   - If a follower becomes leader, it has all committed entries
```

### Where Consensus Is Used

| System | Consensus Protocol | What It Agrees On |
|--------|-------------------|-------------------|
| **etcd** | Raft | Kubernetes cluster state, config, secrets |
| **ZooKeeper** | ZAB (Zookeeper Atomic Broadcast) | Distributed coordination, leader election |
| **Kafka (KRaft)** | Raft (since 3.3+) | Partition leadership, topic metadata |
| **CockroachDB** | Raft | Transaction commit across ranges |
| **Consul** | Raft | Service discovery, health checks, KV store |
| **TiDB/TiKV** | Raft | Distributed transaction commit |

> **Architect's rule:** Choose systems that implement consensus correctly. Never implement your own consensus protocol.

---

## 4. Replication Strategies

### Strategy 1: Leader-Follower (Primary-Replica)

```
                    ┌──────────┐
     Writes ──────→ │  LEADER  │
                    │ (Primary)│
                    └──┬───┬───┘
                       │   │    Replication (sync or async)
              ┌────────┘   └────────┐
              ▼                     ▼
        ┌──────────┐          ┌──────────┐
 Reads ─│ FOLLOWER │   Reads ─│ FOLLOWER │
        │ (Replica)│          │ (Replica)│
        └──────────┘          └──────────┘

✅ Simple, well-understood
✅ Read scaling (add more followers)
✅ No write conflicts (single writer)
❌ Leader is write bottleneck and single point of failure
❌ Failover complexity (promoting a follower)
❌ Replication lag → stale reads from followers

Failover:
  1. Detect leader failure (heartbeat timeout)
  2. Elect new leader (most up-to-date follower)
  3. Reconfigure followers to replicate from new leader
  4. Redirect clients to new leader
  ⚠️ Risk: split-brain if old leader comes back
```

### Strategy 2: Multi-Leader (Active-Active)

```
        ┌──────────┐          ┌──────────┐
 Writes─│ LEADER 1 │←────────→│ LEADER 2 │─Writes
 Reads ─│(Region A)│  Async   │(Region B)│─Reads
        └──────────┘  repl.   └──────────┘

✅ Multi-region writes — low latency for local users
✅ No single point of failure for writes
❌ WRITE CONFLICTS — two leaders modify the same record simultaneously

Conflict Resolution Strategies:
┌──────────────────┬──────────────────────────────────────┐
│ Strategy         │ How                                   │
├──────────────────┼──────────────────────────────────────┤
│ Last-Write-Wins  │ Timestamp-based; latest write wins.   │
│ (LWW)            │ Simple but LOSES DATA silently.       │
├──────────────────┼──────────────────────────────────────┤
│ Application-     │ Show conflict to user, let them       │
│ level merge      │ resolve (like Git merge conflicts).   │
├──────────────────┼──────────────────────────────────────┤
│ CRDTs            │ Data structures that automatically    │
│ (Conflict-free   │ merge without conflicts.              │
│ Replicated Data  │ E.g., counters, sets, registers.      │
│ Types)           │ Used by: Redis, Riak, Figma.          │
└──────────────────┴──────────────────────────────────────┘
```

### Strategy 3: Leaderless (Dynamo-Style)

```
         Client
        ╱  │  ╲
       ╱   │   ╲       Write to W nodes, Read from R nodes
      ▼    ▼    ▼
   ┌────┐┌────┐┌────┐
   │ N1 ││ N2 ││ N3 │   N = 3 (total replicas)
   │ ✅ ││ ✅ ││ ✅ │   W = 2 (write to 2 of 3)
   └────┘└────┘└────┘   R = 2 (read from 2 of 3)

Quorum condition: W + R > N  → guarantees overlap
  W=2 + R=2 = 4 > 3 ✅ → at least one node has latest write

Common configurations:
  N=3, W=2, R=2  → balanced (typical)
  N=3, W=3, R=1  → fast reads, slow writes (read-heavy)
  N=3, W=1, R=3  → fast writes, slow reads (write-heavy)
  N=3, W=1, R=1  → fast but NO consistency guarantee!

Used by: Cassandra, DynamoDB, Riak
```

---

## 5. Partitioning / Sharding

### Key-Range Partitioning

```
Partition 1: A-H    Partition 2: I-P    Partition 3: Q-Z
┌───────────┐       ┌───────────┐       ┌───────────┐
│ Alice     │       │ Ivan      │       │ Quinn     │
│ Bob       │       │ Jane      │       │ Robert    │
│ Carol     │       │ Kate      │       │ Sarah     │
│ ...       │       │ ...       │       │ ...       │
└───────────┘       └───────────┘       └───────────┘

✅ Range queries efficient (find all users A-C)
❌ HOT SPOTS: if many users have names starting with "S",
   Partition 3 is overloaded
```

### Hash Partitioning

```
partition = hash(key) % num_partitions

hash("Alice") % 3 = 0  → Partition 0
hash("Bob")   % 3 = 2  → Partition 2
hash("Carol") % 3 = 1  → Partition 1

✅ Even distribution (no hot spots from key patterns)
❌ Range queries impossible (adjacent keys go to different partitions)
❌ Adding/removing partitions requires rehashing (data movement)
```

### Consistent Hashing

```
          0°
          │
    330°──┼──30°
   ╱      │      ╲
  N3    ──┼──    N1     The ring: 0 to 2^32
  ╲       │       ╱     Nodes placed at hash positions
    270°──┼──90°        Keys placed at hash positions
          │             Key → walk clockwise → first node
        180°
          │
         N2

Adding node N4 at 150°:
  Only keys between N2(180°) and N4(150°) need to move
  Most keys stay on the same node! (minimal disruption)

Used by: Cassandra, DynamoDB, Memcached
```

### Hot Spots and Solutions

| Problem | Solution | How |
|---------|----------|-----|
| Celebrity user (millions of followers) | Key splitting | Append random suffix: `user_123_0`, `user_123_1`, ..., `user_123_9` — spread across 10 partitions |
| Time-series data (today's partition gets all writes) | Compound key | Prefix with sensor ID: `sensor_42_2024-01-15` — distributes across sensors |
| Skewed hash distribution | Virtual nodes | Each physical node owns multiple positions on the hash ring |

---

## 6. Distributed Caching

### Four Caching Strategies

```
1. CACHE-ASIDE (Lazy Loading)                2. READ-THROUGH
─────────────────────────────                ────────────────
   App ──→ Cache hit? ──Yes──→ Return        App ──→ Cache hit? ──Yes──→ Return
    │         │                               │        │
    │        No                               │       No
    │         │                               │        │
    │     App reads DB                        │    Cache reads DB (auto)
    │         │                               │        │
    │     App writes to cache                 │    Cache stores & returns
    │         │                               │
    └─────→ Return                            └──→ Return

  App controls cache logic.                  Cache library handles DB reads.
  Most common pattern.                       Simpler app code.

3. WRITE-THROUGH                             4. WRITE-BEHIND (Write-Back)
──────────────────                           ───────────────────────────
   App ──→ Write to cache                    App ──→ Write to cache ──→ Return (fast!)
              │                                         │
           Cache writes to DB (sync)                 Cache writes to DB (async, batched)
              │                                         │
           Return (slower — two writes)              ⚠️ Data loss risk if cache crashes

  ✅ Cache always up-to-date                 ✅ Very fast writes
  ❌ Higher write latency                    ❌ Data loss risk
  ❌ Cache filled with rarely-read data      ❌ Complex failure handling
```

### Cache-Aside Pattern in Java with Spring Boot + Redis

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository repository;
    private final RedisTemplate<String, Product> redisTemplate;
    private static final Duration CACHE_TTL = Duration.ofMinutes(30);

    public Product getProduct(String productId) {
        String cacheKey = "product:" + productId;
        
        // 1. Try cache first
        Product cached = redisTemplate.opsForValue().get(cacheKey);
        if (cached != null) {
            return cached; // Cache HIT
        }
        
        // 2. Cache MISS — read from database
        Product product = repository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
        
        // 3. Populate cache with TTL
        redisTemplate.opsForValue().set(cacheKey, product, CACHE_TTL);
        
        return product;
    }

    public Product updateProduct(String productId, UpdateProductRequest request) {
        Product product = repository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
        
        product.update(request);
        repository.save(product);
        
        // 4. Invalidate cache (not update — avoids race conditions)
        redisTemplate.delete("product:" + productId);
        
        return product;
    }
}
```

### Redis on Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: caching
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7-alpine
          ports:
            - containerPort: 6379
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          command: ["redis-server", "--maxmemory", "256mb",
                    "--maxmemory-policy", "allkeys-lru"]
---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: caching
spec:
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
```

### Cache Pitfalls

| Pitfall | Description | Solution |
|---------|-------------|---------|
| **Cache Stampede** | Many requests hit cache miss simultaneously, all query DB | Mutex lock: first request locks, others wait; or probabilistic early expiry |
| **Cache Penetration** | Repeated queries for non-existent keys bypass cache, hit DB | Bloom filter to check existence; cache null results with short TTL |
| **Stale Data** | Cached data doesn't reflect latest DB state | TTL-based expiry; event-driven invalidation (CDC or application events) |
| **Memory Pressure** | Cache grows unbounded, OOM kills | Set `maxmemory` + eviction policy (LRU, LFU); monitor hit rate |
| **Hot Key** | Single cache key receives extreme traffic | Replicate hot keys across multiple Redis nodes; local in-process cache |

---

## 7. CQRS Implementation

### Architecture

```
                    ┌──────────────┐
                    │   API Layer  │
                    └──────┬───────┘
                           │
              ┌────────────┴────────────┐
              │                         │
      ┌───────┴────────┐      ┌────────┴───────┐
      │  COMMAND SIDE  │      │   QUERY SIDE   │
      │                │      │                │
      │ CreateOrder    │      │ OrderSummary   │
      │ CancelOrder    │      │ OrderHistory   │
      │ UpdateItem     │      │ OrderSearch    │
      ├────────────────┤      ├────────────────┤
      │ Domain Model   │      │ Denormalized   │
      │ (Order agg.)   │      │ Read Models    │
      ├────────────────┤      ├────────────────┤
      │  PostgreSQL    │      │ Elasticsearch  │
      │  (write DB)    │      │ (read DB)      │
      └───────┬────────┘      └───────▲────────┘
              │                       │
              │    Domain Events      │
              └───────────────────────┘
                  (Kafka / outbox)
```

### Spring Boot CQRS Implementation

```java
// === COMMAND SIDE ===

@Service
@RequiredArgsConstructor
public class OrderCommandService {
    private final OrderRepository repository;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public OrderId createOrder(CreateOrderCommand cmd) {
        Order order = Order.create(cmd.customerId(), cmd.items());
        repository.save(order);
        
        // Publish domain event
        eventPublisher.publishEvent(new OrderCreatedEvent(
            order.getId(), order.getCustomerId(),
            order.getItems(), order.getTotalAmount(),
            Instant.now()
        ));
        return order.getId();
    }

    @Transactional
    public void cancelOrder(CancelOrderCommand cmd) {
        Order order = repository.findById(cmd.orderId())
            .orElseThrow(() -> new OrderNotFoundException(cmd.orderId()));
        order.cancel(cmd.reason());
        repository.save(order);
        
        eventPublisher.publishEvent(new OrderCancelledEvent(
            order.getId(), cmd.reason(), Instant.now()
        ));
    }
}

// === QUERY SIDE ===

@Service
@RequiredArgsConstructor
public class OrderQueryService {
    private final OrderReadRepository readRepository; // Elasticsearch

    public OrderSummaryDto getOrderSummary(String orderId) {
        return readRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
    }

    public Page<OrderSummaryDto> searchOrders(OrderSearchCriteria criteria,
                                               Pageable pageable) {
        return readRepository.search(criteria, pageable);
    }
}

// === PROJECTION (syncs write → read model) ===

@Component
@RequiredArgsConstructor
public class OrderProjection {
    private final OrderReadRepository readRepository;

    @EventListener
    public void on(OrderCreatedEvent event) {
        OrderSummaryDto summary = OrderSummaryDto.builder()
            .orderId(event.orderId().toString())
            .customerId(event.customerId())
            .status("CREATED")
            .totalAmount(event.totalAmount())
            .createdAt(event.occurredAt())
            .items(event.items())
            .build();
        readRepository.save(summary);
    }

    @EventListener
    public void on(OrderCancelledEvent event) {
        readRepository.findById(event.orderId().toString())
            .ifPresent(summary -> {
                summary.setStatus("CANCELLED");
                summary.setCancelledAt(event.occurredAt());
                summary.setCancelReason(event.reason());
                readRepository.save(summary);
            });
    }
}
```

---

## 8. Event Sourcing in Practice

### Event Store Schema

```
┌────────────────────────────────────────────────────────────────┐
│                      event_store table                          │
├──────────┬──────────┬──────────┬───────────┬──────────┬────────┤
│ event_id │ agg_type │ agg_id   │ version   │ event    │ data   │
│ (UUID)   │          │          │ (seq num) │ _type    │ (JSON) │
├──────────┼──────────┼──────────┼───────────┼──────────┼────────┤
│ uuid-1   │ Order    │ ORD-123  │ 1         │ Created  │ {...}  │
│ uuid-2   │ Order    │ ORD-123  │ 2         │ ItemAdd  │ {...}  │
│ uuid-3   │ Order    │ ORD-123  │ 3         │ ItemAdd  │ {...}  │
│ uuid-4   │ Order    │ ORD-123  │ 4         │ Submitted│ {...}  │
│ uuid-5   │ Order    │ ORD-123  │ 5         │ Paid     │ {...}  │
└──────────┴──────────┴──────────┴───────────┴──────────┴────────┘

Unique constraint: (agg_type, agg_id, version) — prevents concurrent modifications

To get current state: replay all events for agg_id in version order
```

### Snapshots for Performance

```
Events:   [1] [2] [3] ... [99] [SNAPSHOT@100] [101] [102] [103]
                                     │
                                     └── Full state at version 100
                                         (serialized aggregate)

To load current state:
  1. Load latest snapshot (version 100)
  2. Load events AFTER snapshot (101, 102, 103)
  3. Apply those 3 events to the snapshot
  Instead of replaying all 103 events!

Rule of thumb: Snapshot every 50-100 events
```

### Event Schema Evolution

| Strategy | How | When |
|----------|-----|------|
| **Upcasting** | Transform old event format to new format when reading | Simple field additions/renames |
| **Versioned Events** | `OrderCreatedV1`, `OrderCreatedV2` — handle both | Significant schema changes |
| **Lazy Migration** | Transform events on read, write back in new format | Gradual background migration |
| **Copy-Transform** | New event store, transform all events in batch | Major restructuring (rare) |

---

## 9. Change Data Capture (CDC)

### Architecture

```
┌───────────┐     ┌──────────────┐     ┌─────────┐     ┌────────────┐
│Application│────→│  PostgreSQL  │────→│Debezium │────→│   Kafka    │
│ (writes)  │     │  (WAL/binlog)│     │Connector│     │  Topics    │
└───────────┘     └──────────────┘     └─────────┘     └──┬───┬───┬─┘
                                                          │   │   │
                                                          ▼   ▼   ▼
                                                      ┌──────────────┐
                                                      │  Consumers:  │
                                                      │ • Search idx │
                                                      │ • Cache inv. │
                                                      │ • Analytics  │
                                                      │ • Audit log  │
                                                      │ • Read models│
                                                      └──────────────┘

Debezium reads the database transaction log (WAL/binlog)
→ no application code changes needed
→ captures ALL changes (even direct SQL updates)
→ exactly-once semantics with Kafka Connect
```

### CDC Use Cases

| Use Case | Description |
|----------|-------------|
| **Search index sync** | Replicate product catalog changes to Elasticsearch |
| **Cache invalidation** | Invalidate Redis cache when DB row changes |
| **Microservice data sync** | Replicate data between service databases |
| **Audit trail** | Capture all data changes for compliance |
| **Real-time analytics** | Stream changes to data warehouse (Snowflake, BigQuery) |

### Tools Comparison

| Tool | Source | Target | Notes |
|------|--------|--------|-------|
| **Debezium** | PostgreSQL, MySQL, MongoDB, SQL Server | Kafka | Open source, most popular, Kafka Connect based |
| **AWS DMS** | Any major DB | RDS, S3, Kinesis, Redshift | Managed service, AWS-native |
| **Azure CDC** | SQL Server, Cosmos DB | Event Hubs, Azure SQL | Azure-native, integrated with Synapse |

---

## 10. Polyglot Persistence

### Database Decision Guide

| Need | Best Fit | Examples |
|------|----------|---------|
| ACID transactions, relational data | Relational DB | PostgreSQL, MySQL, CockroachDB |
| Full-text search, fuzzy matching | Search engine | Elasticsearch, OpenSearch |
| Caching, session store, leaderboards | In-memory KV | Redis, Memcached |
| Document storage, flexible schema | Document DB | MongoDB, Couchbase |
| Wide-column, time-series, high write | Column-family | Cassandra, ScyllaDB, TimescaleDB |
| Graph relationships (social, fraud) | Graph DB | Neo4j, Amazon Neptune |
| Event streams, pub/sub | Event streaming | Kafka, Pulsar, Kinesis |
| File/object storage (images, backups) | Object store | S3, GCS, Azure Blob |

### Service-to-Database Mapping

```
┌──────────────────────────────────────────────────────────┐
│                    E-COMMERCE PLATFORM                     │
│                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Order    │  │ Product  │  │ User     │  │ Analytics│ │
│  │ Service  │  │ Service  │  │ Service  │  │ Service  │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘ │
│       │             │             │              │        │
│  ┌────┴─────┐  ┌────┴─────┐  ┌───┴──────┐  ┌───┴──────┐ │
│  │PostgreSQL│  │ MongoDB  │  │PostgreSQL│  │ ClickHouse│ │
│  │(ACID txn)│  │(flexible │  │(relational│ │(columnar  │ │
│  │          │  │ catalog) │  │ + auth)  │  │ analytics)│ │
│  └──────────┘  └────┬─────┘  └──────────┘  └──────────┘ │
│                     │                                     │
│                ┌────┴──────┐                              │
│                │Elastic    │                              │
│                │Search     │                              │
│                │(full-text)│                              │
│                └───────────┘                              │
│                                                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │               Redis (caching layer)               │    │
│  └──────────────────────────────────────────────────┘    │
│                                                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │            Kafka (event backbone)                 │    │
│  └──────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

## 11. Exercises

### Exercise 1: Consistency Model Design

For an e-commerce platform, specify the consistency model for each feature and justify:

1. **Product price display** on catalog page
2. **Inventory count** during checkout
3. **User's shopping cart** across devices
4. **Order confirmation** after payment
5. **Product review count** on product page
6. **User profile** changes (name, email)

For each: Name the consistency model, explain why, and describe the implementation approach.

### Exercise 2: Sharding Strategy

You have 500 million user records in a social media platform. Design a sharding strategy:

1. What is your shard key? Why?
2. How many shards and what partitioning method?
3. How do you handle the "celebrity user" hot spot problem?
4. How do you query "all friends of user X" when friends are on different shards?
5. How do you handle shard rebalancing when adding capacity?

### Exercise 3: Caching Architecture

Design a caching architecture for a product catalog service (10M products, 100K requests/second, 95% reads):

1. Which caching strategy (cache-aside, read-through, etc.) and why?
2. What is your cache key design?
3. How do you handle cache invalidation when products are updated?
4. How do you prevent cache stampede on popular products?
5. What is your eviction policy and TTL strategy?
6. Draw the full architecture diagram.

### Exercise 4: CQRS for Order Management

Design a CQRS architecture for an order management system with:
- Write side: order creation, modification, cancellation (complex business rules)
- Read side: order dashboard, searchable history, real-time tracking

1. Define the command and query APIs
2. Choose databases for write and read sides
3. Define the events that sync write to read
4. Design the read model schema (denormalized)
5. How do you handle the consistency delay between write and read?

---

## 12. Self-Check Questions

1. Explain the CAP theorem. Why is "pick two" misleading?
2. What is the PACELC theorem and why is it more useful than CAP?
3. Compare strong consistency and eventual consistency. When do you choose each?
4. Explain the Raft consensus algorithm at a high level. Where is it used?
5. What are the three replication strategies? Compare tradeoffs.
6. Explain consistent hashing and why it's useful for partitioning.
7. Describe the cache-aside pattern. How do you prevent cache stampede?
8. What is CDC and what problem does it solve?

---

## 13. Answers

### Answer 1
CAP theorem states that a distributed system can provide at most two of: Consistency (linearizability), Availability (every non-failing node responds), Partition tolerance (operates despite network partitions). "Pick two" is misleading because network partitions are unavoidable in distributed systems — you don't "choose" partition tolerance, you must have it. The real choice is: during a partition, do you sacrifice consistency (AP) or availability (CP)? During normal operation (no partition), you can have both C and A.

### Answer 2
PACELC extends CAP: if there's a Partition, choose Availability or Consistency; Else (normal operation), choose Latency or Consistency. It's more useful because most of the time systems operate without partitions, and the latency-consistency tradeoff matters more in practice. For example, Cassandra is PA/EL (available during partitions, low latency normally), while CockroachDB is PC/EC (consistent always, higher latency due to consensus).

### Answer 3
**Strong consistency:** Every read sees the latest write. Implementation: synchronous replication or consensus protocol. Choose for: financial transactions, inventory management, anything where stale data causes business errors. Cost: higher latency. **Eventual consistency:** Reads may return stale data, but all replicas converge. Choose for: social media counters, product reviews, DNS — where slight staleness is acceptable and low latency matters more.

### Answer 4
Raft elects a leader from a set of nodes using majority votes. The leader accepts all writes, replicates them to followers. A write is "committed" when a majority of nodes acknowledge it. If the leader fails, a new election occurs. Used in: etcd (Kubernetes state), Kafka KRaft (metadata), CockroachDB (transaction commit), Consul (service discovery). Key rule: never implement consensus yourself — use systems that have proven implementations.

### Answer 5
(1) **Leader-Follower:** Single leader accepts writes, followers replicate. Simple, no conflicts, but leader is bottleneck. (2) **Multi-Leader:** Multiple leaders accept writes, replicate to each other. Low write latency multi-region, but write conflicts need resolution (LWW, CRDTs). (3) **Leaderless (Dynamo):** Write to W nodes, read from R nodes. If W+R>N, guaranteed to read latest. Highly available, but more complex conflict handling.

### Answer 6
Consistent hashing places nodes and keys on a circular hash ring (0 to 2^32). A key is stored on the first node clockwise from its position. When a node is added/removed, only keys between the new/removed node and its predecessor need to move — most data stays in place. Without consistent hashing, adding a node would require rehashing and moving a large portion of all data. Virtual nodes (multiple positions per physical node) improve balance.

### Answer 7
Cache-aside: Application checks cache first. On miss, reads from DB, stores in cache, returns. On write, updates DB, then invalidates (deletes) cache entry. **Cache stampede prevention:** (a) Mutex/lock: first request acquires lock, others wait for it to populate cache. (b) Probabilistic early expiry: randomly refresh before TTL expires. (c) Background refresh: separate process refreshes cache before expiry.

### Answer 8
Change Data Capture captures row-level changes from a database's transaction log (PostgreSQL WAL, MySQL binlog) and streams them to consumers (typically via Kafka). It solves: (a) keeping read models and search indexes in sync without application code changes, (b) building event-driven pipelines from legacy databases, (c) cross-service data synchronization in microservices, (d) real-time analytics from operational databases. Debezium is the most popular open-source CDC tool.

---

## References

- "Designing Data-Intensive Applications" — Martin Kleppmann
- [Debezium](https://debezium.io/) — Change Data Capture
- [Redis Documentation](https://redis.io/docs/)
- [Raft Consensus Algorithm](https://raft.github.io/)
- [Martin Fowler — CQRS](https://martinfowler.com/bliki/CQRS.html)

---

*Previous lesson: [03 — Microservices Deep Dive](03-microservices-deep-dive.md)*  
*Next lesson: [05 — Containers & Orchestration](05-containers-orchestration.md)*