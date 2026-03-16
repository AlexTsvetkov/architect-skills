# Lesson 10: Event-Driven Architecture & Messaging

> **Prerequisites:** Lessons 01-09  
> **Time estimate:** 8-10 hours  
> **Parallel reading:** DDIA — stream processing chapters (Ch 11-12)

---

## Table of Contents

1. [Event-Driven Patterns](#1-event-driven-patterns)
2. [Apache Kafka Deep Dive](#2-apache-kafka-deep-dive)
3. [Event Schema Evolution](#3-event-schema-evolution)
4. [Reliability Patterns](#4-reliability-patterns)
5. [Stream Processing](#5-stream-processing)
6. [Exercises](#6-exercises)
7. [Self-Check Questions](#7-self-check-questions)
8. [Answers](#8-answers)

---

## 1. Event-Driven Patterns

### Three Event Patterns

| Pattern | Description | Event Contains | Use When |
|---------|-------------|---------------|----------|
| **Event Notification** | Something happened | Minimal: `{ orderId: 123 }` | Consumers can fetch details if needed |
| **Event-Carried State Transfer** | Something happened + all data | Full: `{ orderId: 123, items: [...], customer: {...} }` | Consumers need to be autonomous |
| **Event Sourcing** | State changes as events | State delta: `ItemAdded { productId, qty }` | Need full audit trail |

### Message Brokers Comparison

| Broker | Model | Ordering | Retention | Best For |
|--------|-------|----------|-----------|----------|
| Apache Kafka | Log-based | Per partition | Days/forever | High throughput, event sourcing, streaming |
| RabbitMQ | Queue-based | Per queue | Until consumed | Task queues, RPC, routing |
| AWS SQS/SNS | Cloud queue + pub/sub | Best-effort | 14 days | AWS-native, simple queuing |
| Azure Service Bus | Enterprise messaging | Per session | Configurable | Azure-native, enterprise patterns |

---

## 2. Apache Kafka Deep Dive

### Architecture

```
┌──────────────── KAFKA CLUSTER ─────────────────┐
│                                                  │
│  Topic: orders (3 partitions)                    │
│                                                  │
│  Partition 0: [msg0][msg3][msg6][msg9]          │
│  Partition 1: [msg1][msg4][msg7][msg10]         │
│  Partition 2: [msg2][msg5][msg8][msg11]         │
│                                                  │
│  Broker 1 (P0-leader, P1-replica)               │
│  Broker 2 (P1-leader, P2-replica)               │
│  Broker 3 (P2-leader, P0-replica)               │
└──────────────────────────────────────────────────┘

Producer ──→ Partition (by key hash or round-robin)
Consumer Group ──→ Each partition assigned to one consumer
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Topic** | Named category of messages (like a DB table) |
| **Partition** | Ordered, immutable log within a topic |
| **Offset** | Position of a message within a partition |
| **Consumer Group** | Set of consumers that share the work of consuming a topic |
| **Replication Factor** | Number of copies of each partition across brokers |

### Ordering Guarantees

```
Messages with same key → same partition → guaranteed order
Messages without key → round-robin → no order guarantee

Example:
  Key = orderId
  All events for order-123 go to the same partition
  → OrderCreated, ItemAdded, PaymentProcessed arrive in order
```

### Consumer Groups

```
Topic: orders (3 partitions)

Consumer Group A (Order Processing):
  Consumer A1 ← Partition 0
  Consumer A2 ← Partition 1, Partition 2

Consumer Group B (Analytics):
  Consumer B1 ← Partition 0, Partition 1, Partition 2

Each group gets ALL messages (independent processing).
Within a group, each partition is consumed by ONE consumer.
Max consumers in a group = number of partitions.
```

### Exactly-Once Semantics

```
At-most-once:  May lose messages. Fire and forget.
At-least-once: May get duplicates. Ack after processing.
Exactly-once:  No loss, no duplicates.

Kafka provides exactly-once with:
  1. Idempotent producers (dedup at broker)
  2. Transactional producers (atomic writes across partitions)
  3. Consumer read-process-write transactions

In practice: use at-least-once + idempotent consumers (simpler, more common).
```

### Spring Boot with Kafka

```java
// Producer
@Service
public class OrderEventPublisher {
    @Autowired private KafkaTemplate<String, OrderEvent> kafka;

    public void publishOrderCreated(Order order) {
        OrderCreated event = new OrderCreated(order.getId(), order.getItems());
        kafka.send("orders", order.getId(), event);  // key = orderId
    }
}

// Consumer
@Component
public class OrderEventConsumer {
    @KafkaListener(topics = "orders", groupId = "payment-service")
    public void handleOrderCreated(OrderCreated event) {
        paymentService.processPayment(event.getOrderId(), event.getTotal());
    }
}
```

---

## 3. Event Schema Evolution

### The Problem

Events are stored forever (or for a long time). Schemas change over time. Old consumers must handle new events, and new consumers must handle old events.

### Compatibility Rules

| Compatibility | Description | Safe Changes |
|--------------|-------------|-------------|
| **Backward** | New schema can read old data | Add optional fields, remove fields with defaults |
| **Forward** | Old schema can read new data | Remove optional fields, add fields with defaults |
| **Full** | Both backward and forward | Add optional fields with defaults only |

### Schema Registry (Confluent)

```
Producer → Schema Registry: "Register schema v2 for topic orders"
Registry: "v2 is backward-compatible with v1 ✅" (or rejects)
Producer → Kafka: send message with schema ID
Consumer → Schema Registry: "Give me schema for ID 42"
Consumer: deserialize message using retrieved schema
```

### Avro Schema Evolution Example

```json
// v1
{
  "type": "record",
  "name": "OrderCreated",
  "fields": [
    {"name": "orderId", "type": "string"},
    {"name": "total", "type": "double"}
  ]
}

// v2 — backward compatible (added optional field)
{
  "type": "record",
  "name": "OrderCreated",
  "fields": [
    {"name": "orderId", "type": "string"},
    {"name": "total", "type": "double"},
    {"name": "currency", "type": "string", "default": "USD"}
  ]
}
```

---

## 4. Reliability Patterns

### Transactional Outbox

```
Problem: Dual write — update DB and publish event atomically.

Solution:
BEGIN TRANSACTION;
  INSERT INTO orders (id, ...) VALUES (...);
  INSERT INTO outbox (id, event_type, payload) VALUES (...);
COMMIT;

Outbox Relay (polling or CDC):
  SELECT * FROM outbox WHERE published = false;
  → Publish to Kafka
  → UPDATE outbox SET published = true;
```

### Dead Letter Queue (DLQ)

```
Main Queue → Consumer → Process
                │
                ├─ Success → Ack message
                └─ Failure (after N retries) → Dead Letter Queue
                                                    │
                                              Manual review
                                              or automated retry later
```

### Idempotent Consumers

```java
@KafkaListener(topics = "payments")
public void handlePayment(PaymentEvent event) {
    // Check if already processed
    if (processedEvents.contains(event.getEventId())) {
        log.info("Duplicate event {}, skipping", event.getEventId());
        return;
    }
    
    // Process
    paymentService.process(event);
    
    // Mark as processed
    processedEvents.add(event.getEventId());
}
```

### Poison Message Handling

Messages that consistently fail to process (bad format, invalid data). Without handling, they block the consumer forever.

Solutions:
1. Dead Letter Queue after N retries
2. Skip and log after N retries
3. Schema validation before processing

---

## 5. Stream Processing

### Kafka Streams vs. Apache Flink

| Aspect | Kafka Streams | Apache Flink |
|--------|-------------|--------------|
| Deployment | Library (runs in your app) | Separate cluster |
| Complexity | Simple | Complex but powerful |
| State management | RocksDB (local) | Managed state backends |
| Windowing | Basic | Advanced (event-time, watermarks) |
| Best for | Simple transformations, aggregations | Complex event processing, ML |

### Common Stream Operations

```
Filter:     orders.filter(o -> o.getTotal() > 100)
Map:        orders.mapValues(o -> new OrderSummary(o))
Aggregate:  orders.groupByKey().count()
Join:       orders.join(payments, (o, p) -> new OrderWithPayment(o, p))
Window:     orders.windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(5)))
                  .count()
```

---

## 6. Exercises

### Exercise 1: Event Design

Design the event architecture for an e-commerce order flow:
1. List all events (at least 10)
2. For each: producer, consumers, payload schema
3. Choose event pattern (notification vs. state transfer) per event
4. Design Kafka topic structure (how many topics, partition keys)

### Exercise 2: Failure Handling

Design a failure handling strategy:
1. What happens when a consumer fails to process a message?
2. How do you handle poison messages?
3. How do you ensure idempotency?
4. How do you handle out-of-order messages?

### Exercise 3: Schema Evolution

You have an `OrderCreated` event consumed by 5 services. You need to:
- Add a `loyaltyPoints` field
- Change `total` from `double` to a `Money` object `{amount, currency}`
- Remove the deprecated `legacyId` field

Plan the migration: which changes are safe, which need versioning, and the rollout sequence.

---

## 7. Self-Check Questions

1. Explain the three event patterns and when to use each.
2. How do Kafka partitions and consumer groups work together?
3. What is the transactional outbox pattern and what problem does it solve?
4. Why are idempotent consumers important?
5. What is schema evolution and why is backward compatibility important?
6. Compare Kafka and RabbitMQ — when would you use each?

---

## 8. Answers

### Answer 1
**Event Notification:** Minimal data (`{orderId: 123}`). Consumer calls back to source for details. Use when: not all consumers need the data, reduces event size, but adds runtime coupling. **Event-Carried State Transfer:** Full data included. Consumer processes autonomously without callbacks. Use when: consumers need independence, multiple consumers need same data, reduce latency. **Event Sourcing:** State changes stored as sequence of events. Current state = replay all events. Use when: need audit trail, temporal queries, complex domain state transitions.

### Answer 2
A topic is split into partitions — ordered logs. Producers send messages to partitions (by key hash or round-robin). Within a consumer group, each partition is assigned to exactly one consumer. This means: max parallelism = number of partitions. If a consumer dies, its partitions are reassigned to remaining consumers (rebalancing). Multiple consumer groups can read the same topic independently — each group gets all messages.

### Answer 3
The transactional outbox solves the dual-write problem: you need to update a database AND publish an event, but can't do both atomically. If the DB write succeeds but Kafka publish fails, data is inconsistent. Solution: write both the data change and the event to the same database in one transaction. A separate relay process reads the outbox table and publishes to Kafka. Guarantees at-least-once event delivery because the event is persisted in the same transaction as the data.

### Answer 4
In distributed systems, messages may be delivered more than once (network retries, consumer crashes after processing but before ack). If the consumer is not idempotent, processing a duplicate message causes incorrect results (double charge, duplicate order). Idempotent consumers produce the same result regardless of how many times a message is processed. Implementation: store processed event IDs, use database upsert instead of insert, use idempotency keys.

### Answer 5
Schema evolution is changing event structure over time while maintaining compatibility. **Backward compatibility** means new consumers can read old events — critical because old events exist in Kafka for days/weeks/forever. Without it, deploying a new consumer version would fail on old messages. Achieved by: only adding optional fields with defaults, using schema registry to enforce compatibility checks before allowing schema changes.

### Answer 6
**Kafka:** Log-based, messages retained after consumption, replay possible, high throughput, ordering per partition. Best for: event sourcing, event streaming, high-volume data pipelines, when multiple consumers need the same messages. **RabbitMQ:** Queue-based, messages removed after consumption, supports complex routing (exchanges, bindings), lower latency per message. Best for: task queues, RPC patterns, complex routing rules, when you need message acknowledgment and redelivery.

---

*Previous: [09 - Security](09-security.md)*  
*Next: [11 - CI/CD & GitOps](11-cicd-gitops.md)*