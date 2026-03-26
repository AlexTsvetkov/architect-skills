# Prompt: Lesson 10 — Event-Driven Architecture & Messaging

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 10: Event-Driven Architecture & Messaging**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot with Spring Kafka. Builds on DDIA stream processing chapters.

### Topic Coverage

1. **Event-Driven Patterns**
   - Three event patterns table (Event Notification, Event-Carried State Transfer, Event Sourcing)
   - Message brokers comparison table (Kafka, RabbitMQ, AWS SQS/SNS, Azure Service Bus)

2. **Apache Kafka Deep Dive**
   - Architecture diagram (cluster with 3 partitions across 3 brokers)
   - Key concepts table (Topic, Partition, Offset, Consumer Group, Replication Factor)
   - Ordering guarantees (key-based partitioning)
   - Consumer groups diagram (two groups reading same topic independently)
   - Exactly-once semantics (idempotent producers, transactional producers, practical: at-least-once + idempotent consumers)
   - Spring Boot Kafka producer and consumer code

3. **Event Schema Evolution**
   - The problem: events stored forever, schemas change
   - Compatibility rules table (Backward, Forward, Full)
   - Schema Registry workflow (producer registers, consumer fetches)
   - Avro schema evolution example (v1 → v2 with optional field)

4. **Reliability Patterns**
   - Transactional Outbox (DB transaction + outbox table + relay)
   - Dead Letter Queue (DLQ) flow diagram
   - Idempotent consumers (Java code with processed event tracking)
   - Poison message handling (3 solutions)

5. **Stream Processing**
   - Kafka Streams vs Apache Flink comparison table
   - Common stream operations code (filter, map, aggregate, join, window)

### Code Examples Required
1. Kafka producer with Spring Boot (KafkaTemplate, key-based sending)
2. Kafka consumer with @KafkaListener
3. Idempotent consumer with duplicate detection
4. Transactional outbox SQL pattern
5. Avro schema v1 and v2 JSON

### Diagrams Required (minimum 6)
Kafka cluster architecture, Consumer groups, Transactional outbox flow, DLQ flow, Schema registry workflow, Event-carried state transfer vs notification

### Real-World Case Study
Design event architecture for e-commerce order flow: 10+ events, producers/consumers, Kafka topic structure, failure handling strategy.

### Exercises (3)
1. Event design for order flow (10 events, schemas, topics, partition keys)
2. Failure handling strategy (consumer failure, poison messages, idempotency, out-of-order)
3. Schema evolution migration plan (add field, change type, remove field)

### Self-Check Questions (6)
Cover: three event patterns, Kafka partitions + consumer groups, transactional outbox, idempotent consumers, schema evolution + backward compatibility, Kafka vs RabbitMQ.

### References
Include: "Designing Data-Intensive Applications" (Kleppmann), Confluent Kafka documentation, Spring Kafka docs, "Enterprise Integration Patterns" (Hohpe & Woolf), Microsoft event-driven architecture guide.