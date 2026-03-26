# Lesson 06: Cloud-Native Design Patterns

> **Prerequisites:** Lessons 01-05  
> **Time estimate:** 6-8 hours  
> **Parallel reading:** "Cloud Native Patterns" — Cornelia Davis, "Release It!" — Michael Nygard

---

## Table of Contents

1. [Resilience Patterns](#1-resilience-patterns)
2. [Structural Patterns](#2-structural-patterns)
3. [Migration Patterns](#3-migration-patterns)
4. [Communication Patterns](#4-communication-patterns)
5. [Exercises](#5-exercises)
6. [Self-Check Questions](#6-self-check-questions)
7. [Answers](#7-answers)

---

## 1. Resilience Patterns

### Circuit Breaker

Prevent cascading failures by stopping calls to a failing service.

```
States:
  CLOSED ──(failure threshold reached)──→ OPEN
  OPEN ──(timeout expires)──→ HALF-OPEN
  HALF-OPEN ──(test request succeeds)──→ CLOSED
  HALF-OPEN ──(test request fails)──→ OPEN

CLOSED: Requests flow normally. Failures counted.
OPEN: Requests immediately fail (fast-fail). No calls to failing service.
HALF-OPEN: Limited test requests to check if service recovered.
```

**Spring Boot with Resilience4j:**
```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public PaymentResult processPayment(PaymentRequest request) {
    return paymentClient.charge(request);
}

public PaymentResult paymentFallback(PaymentRequest request, Exception e) {
    return PaymentResult.queued("Payment queued for retry");
}
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
```

### Retry with Exponential Backoff and Jitter

```
Attempt 1: immediate
Attempt 2: wait 1s + random(0-500ms)
Attempt 3: wait 2s + random(0-500ms)
Attempt 4: wait 4s + random(0-500ms)
Attempt 5: give up

Jitter prevents "thundering herd" — all retries hitting service simultaneously.
```

```java
@Retry(name = "inventoryService", fallbackMethod = "inventoryFallback")
public StockLevel checkInventory(String productId) {
    return inventoryClient.getStock(productId);
}
```

### Bulkhead

Isolate failures so one failing component doesn't consume all resources.

```
Without bulkhead:
  Thread Pool (20 threads)
  ├── Payment Service calls: 15 (slow, timing out)
  ├── Inventory Service calls: 3
  └── Email Service calls: 2
  → Payment problems starve Inventory and Email!

With bulkhead (thread pool isolation):
  Payment Pool (8 threads): 8 (slow, only uses its allocation)
  Inventory Pool (6 threads): 3 
  Email Pool (6 threads): 2
  → Payment problems contained to its pool
```

```java
@Bulkhead(name = "paymentService", type = Bulkhead.Type.THREADPOOL)
public PaymentResult processPayment(PaymentRequest request) {
    return paymentClient.charge(request);
}
```

### Timeout

Always set timeouts. Never wait forever.

```java
@TimeLimiter(name = "paymentService")
public CompletableFuture<PaymentResult> processPayment(PaymentRequest req) {
    return CompletableFuture.supplyAsync(() -> paymentClient.charge(req));
}
```

```yaml
resilience4j:
  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 3s
```

### Health Endpoint Monitoring

```java
// Spring Boot Actuator
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        if (isDatabaseReachable()) {
            return Health.up().withDetail("db", "PostgreSQL reachable").build();
        }
        return Health.down().withDetail("db", "Connection failed").build();
    }
}

// Kubernetes uses these endpoints:
// /actuator/health/liveness  → restart pod if unhealthy
// /actuator/health/readiness → remove from load balancer if not ready
```

### Rate Limiting / Throttling

```java
@RateLimiter(name = "apiRateLimit", fallbackMethod = "rateLimitFallback")
public Response handleRequest(Request request) {
    return processRequest(request);
}

public Response rateLimitFallback(Request request, Exception e) {
    return Response.status(429).body("Too many requests. Retry after 60s.");
}
```

---

## 2. Structural Patterns

### Sidecar Pattern

Deploy a helper container alongside the main application container in the same pod.

```
┌─────────────────────────── Pod ──────────────────────────┐
│                                                           │
│  ┌──────────────────┐     ┌──────────────────────┐       │
│  │  Main Container   │     │  Sidecar Container    │       │
│  │  (App)            │     │  (Log Collector /     │       │
│  │                   │     │   Envoy Proxy /       │       │
│  │  Order Service    │←───→│   Config Reloader)    │       │
│  │  (Java)           │     │                       │       │
│  └──────────────────┘     └──────────────────────┘       │
│                                                           │
│  Shared: network (localhost), storage (volumes)           │
└───────────────────────────────────────────────────────────┘

Examples:
- Envoy proxy (Istio service mesh)
- Fluentd/Filebeat (log collection)
- Vault Agent (secret injection)
- Config reloader (watch ConfigMap changes)
```

### Ambassador Pattern

A special sidecar that proxies outbound network traffic.

```
App → Ambassador (localhost) → External Service

Ambassador handles: retries, circuit breaking, TLS, routing
App doesn't need to implement these concerns.
```

### Adapter Pattern

A sidecar that standardizes the interface of the main container.

```
Monitoring System (expects Prometheus format)
      ↑
┌─────┴──────────────── Pod ──────────────────┐
│  ┌─────────────┐    ┌──────────────────┐    │
│  │  Legacy App  │───→│  Adapter Sidecar  │    │
│  │  (custom     │    │  (converts to    │    │
│  │   metrics)   │    │   Prometheus     │    │
│  │              │    │   format)        │    │
│  └─────────────┘    └──────────────────┘    │
└──────────────────────────────────────────────┘
```

---

## 3. Migration Patterns

### Strangler Fig Pattern

Gradually replace monolith functionality with microservices.

```
Phase 1: Intercept → Route → Monolith/New Service
Phase 2: Migrate more routes → New Services grow, Monolith shrinks
Phase 3: Monolith fully replaced

Implementation:
- API Gateway / reverse proxy as the interceptor
- Route by URL path, header, or percentage
- Event-driven: new service subscribes to monolith DB changes (CDC)
```

### Branch by Abstraction

Refactor internal code while keeping the system running.

```
Step 1: Identify the code to replace (e.g., NotificationService)
Step 2: Create an abstraction layer (interface)
Step 3: Make existing code implement the interface
Step 4: Build new implementation behind the interface
Step 5: Switch to new implementation (feature flag)
Step 6: Remove old implementation

// Step 2-3: Abstraction
interface NotificationSender {
    void send(Notification n);
}

class LegacyNotificationSender implements NotificationSender { ... }

// Step 4: New implementation
class MicroserviceNotificationSender implements NotificationSender { ... }

// Step 5: Feature flag
@Bean NotificationSender notificationSender() {
    return featureFlags.isEnabled("new-notifications")
        ? new MicroserviceNotificationSender(restClient)
        : new LegacyNotificationSender(emailService);
}
```

### Anti-Corruption Layer (ACL)

Protect your new service from the legacy system's model.

```
┌──────────────┐     ┌─────────────────────┐     ┌──────────────┐
│  New Service  │────→│ Anti-Corruption Layer│────→│ Legacy System │
│  (clean model)│     │ (translates between  │     │ (messy model) │
│               │     │  new and old models)  │     │               │
└──────────────┘     └─────────────────────┘     └──────────────┘

ACL translates:
- Data formats (new JSON ↔ old XML)
- Naming conventions (customerId ↔ KUNNR)
- Business concepts (Order ↔ SalesDocument)
```

---

## 4. Communication Patterns

### Gateway Aggregation

Single API call that fans out to multiple backend services.

```
Client: GET /dashboard
  → Gateway:
    1. GET /orders (Order Service)
    2. GET /notifications (Notification Service)
    3. GET /recommendations (Rec Service)
  ← Aggregate responses into single response
```

### Competing Consumers

Multiple consumers process messages from the same queue for parallel processing.

```
┌──────────┐     ┌──────────┐
│ Producer  │────→│  Queue   │
└──────────┘     └────┬─────┘
                      │
          ┌───────────┼───────────┐
          │           │           │
     ┌────┴───┐  ┌───┴────┐  ┌──┴─────┐
     │Worker 1│  │Worker 2│  │Worker 3│
     └────────┘  └────────┘  └────────┘

Each message consumed by exactly ONE worker.
Scale by adding more workers.
```

### Transactional Outbox

Reliably publish events when you change data.

```
❌ Problem: Write to DB then publish event — if publish fails, data is inconsistent.

✅ Solution: Write data AND event to same DB in one transaction.
   Separate process reads outbox table and publishes events.

BEGIN TRANSACTION;
  INSERT INTO orders (...);
  INSERT INTO outbox (event_type, payload) VALUES ('OrderCreated', '...');
COMMIT;

Outbox Publisher (polling or CDC):
  Reads outbox table → publishes to Kafka → marks as published
```

### Valet Key Pattern

Give clients direct access to a resource using a temporary token.

```
1. Client → API: "I want to upload a large file"
2. API → Storage: "Generate a pre-signed upload URL"
3. API → Client: "Upload directly to this URL: https://s3.../upload?token=..."
4. Client → S3: Upload file directly (no load on API server)
```

---

## 5. Exercises

### Exercise 1: Resilience Design

Design a resilience strategy for a payment processing microservice:
1. Which patterns would you apply? (circuit breaker, retry, bulkhead, timeout)
2. What are the configuration values and why?
3. What fallback behavior would you implement?
4. How would you test the resilience? (chaos engineering scenarios)

### Exercise 2: Migration Plan

You have a large Spring Boot e-commerce monolith (500K+ LOC, single PostgreSQL database, 15 interconnected modules). Design a migration plan using cloud-native patterns:
1. Which pattern would you use for incremental migration?
2. What would you extract first and why?
3. How would you handle the shared database?
4. How would you handle data synchronization during migration?

### Exercise 3: Pattern Combination

Design an architecture for a food delivery platform using at least 5 patterns from this lesson. For each pattern, explain why you chose it and where it applies.

---

## 6. Self-Check Questions

1. Explain the three states of a circuit breaker and the transitions between them.
2. Why is jitter important in retry with exponential backoff?
3. What is the bulkhead pattern and what problem does it solve?
4. Explain the sidecar pattern. Give three examples of sidecars.
5. How does the strangler fig pattern work for monolith migration?
6. What is the transactional outbox pattern and what problem does it solve?
7. When would you use the BFF (Backend for Frontend) pattern?

---

## 7. Answers

### Answer 1
**CLOSED:** Normal operation. All requests pass through to the service. Failures are counted within a sliding window. When the failure rate exceeds the threshold (e.g., 50%), transitions to OPEN.

**OPEN:** All requests immediately fail without calling the service (fast-fail). A fallback response is returned. After a configured timeout (e.g., 30 seconds), transitions to HALF-OPEN.

**HALF-OPEN:** A limited number of test requests (e.g., 3) are allowed through. If they succeed, transitions back to CLOSED (service recovered). If they fail, transitions back to OPEN (still failing).

### Answer 2
Without jitter, if a service goes down and 1000 clients all start retrying with the same exponential backoff schedule, they all retry at the exact same times (1s, 2s, 4s, 8s). This creates "thundering herd" — periodic waves of traffic that can overwhelm the recovering service. Jitter adds a random delay (e.g., 0-500ms) to each retry, spreading out the retries over time and giving the service a chance to recover.

### Answer 3
The bulkhead pattern isolates resources (thread pools, connection pools) for different components so that one failing component doesn't exhaust all shared resources. Named after ship bulkheads that contain flooding to one compartment. Problem: without bulkheads, a slow payment service could consume all 200 threads in the shared pool, making inventory and order services unavailable too. With bulkheads: payment gets 50 threads, inventory gets 75, orders get 75. Payment problems only affect its own allocation.

### Answer 4
The sidecar pattern deploys a helper container alongside the main application container in the same Kubernetes pod. They share the network (localhost) and can share storage volumes.

Examples: (1) **Envoy proxy** — handles mTLS, traffic management, observability for service mesh. (2) **Fluentd/Filebeat** — collects log files from the app container and ships them to centralized logging. (3) **Vault Agent** — fetches secrets from HashiCorp Vault and injects them as files/env vars.

### Answer 5
Strangler fig migration: (1) Place a routing layer (API gateway) in front of the monolith. (2) Implement new functionality or rewrite one piece as a microservice. (3) Route that specific traffic to the new service. (4) Repeat — each iteration extracts more functionality. (5) Monolith gradually shrinks until fully replaced. Benefits: incremental, low risk, continuous delivery of value, easy rollback per feature.

### Answer 6
The transactional outbox solves the dual-write problem: you need to update your database AND publish an event, but you can't do both atomically (DB commit might succeed but Kafka publish fails). Solution: write the event to an "outbox" table in the same database transaction as your data change. A separate process (polling or CDC) reads the outbox table and publishes events to Kafka. This guarantees at-least-once delivery because both the data and the event are in the same transaction.

### Answer 7
Use BFF when: different frontends (mobile, web, 3rd party) need different API shapes, payloads, or optimization. Mobile needs smaller payloads and fewer round trips; web needs richer data; 3rd party needs versioned, rate-limited access. Each frontend team owns their BFF, enabling independent evolution. Don't use BFF when you have a single frontend or all frontends need the same data.

---

*Previous: [05 - Containers & Orchestration](05-containers-orchestration.md)*  
*Next: [07 - API Design & Management](07-api-design.md)*