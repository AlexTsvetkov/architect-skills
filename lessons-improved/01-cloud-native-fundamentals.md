# Lesson 01: Cloud-Native Architecture Fundamentals

> **Prerequisites:** Java development experience, basic understanding of web applications  
> **Time estimate:** 4-6 hours  
> **Parallel reading:** "Fundamentals of Software Architecture" — Richards & Ford (Part I)

---

## Table of Contents

1. [What is Cloud-Native?](#1-what-is-cloud-native)
2. [The 12-Factor App Methodology](#2-the-12-factor-app-methodology)
3. [Beyond 12-Factor: The 15-Factor App](#3-beyond-12-factor-the-15-factor-app)
4. [Cloud-Native Maturity Model](#4-cloud-native-maturity-model)
5. [The CNCF Landscape](#5-the-cncf-landscape)
6. [Evolution: Monolith → SOA → Microservices → Serverless](#6-evolution-monolith--soa--microservices--serverless)
7. [Cloud-Native vs. Cloud-Hosted vs. Cloud-Ready](#7-cloud-native-vs-cloud-hosted-vs-cloud-ready)
8. [Exercises](#8-exercises)
9. [Self-Check Questions](#9-self-check-questions)
10. [Answers](#10-answers)

---

## 1. What is Cloud-Native?

### The CNCF Definition

The Cloud Native Computing Foundation (CNCF) defines cloud-native as:

> *"Cloud-native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach."*

### What This Really Means

Cloud-native is **not** just "running your app in the cloud." It is a set of architectural principles and practices that:

1. **Embrace the cloud's elasticity** — design for horizontal scaling, not bigger servers
2. **Assume failure** — every component can and will fail; design for resilience
3. **Automate everything** — from deployment to recovery to scaling
4. **Optimize for speed of change** — small, frequent, safe deployments

### The Four Pillars of Cloud-Native

```
┌─────────────────────────────────────────────────────────────┐
│                      CLOUD-NATIVE                            │
├───────────────┬───────────────┬──────────────┬──────────────┤
│ MICROSERVICES │  CONTAINERS   │    CI/CD     │    DEVOPS    │
│               │               │              │              │
│ Small,        │ Lightweight,  │ Automated    │ Culture of   │
│ independent   │ portable      │ build, test, │ shared       │
│ services with │ runtime       │ deploy       │ ownership    │
│ single        │ environment   │ pipelines    │ between dev  │
│ responsibility│ (Docker, OCI) │ (continuous) │ and ops      │
│               │               │              │              │
│ Own data,     │ Immutable     │ Fast         │ "You build   │
│ own lifecycle │ images        │ feedback     │ it, you      │
│               │               │ loops        │ run it"      │
└───────────────┴───────────────┴──────────────┴──────────────┘
```

### Key Principles

| Principle | Description | Enterprise Monolith Context |
|-----------|-------------|----------------------------|
| **Design for failure** | Assume any component can crash. Build in retries, circuit breakers, fallbacks. | Single-point-of-failure monolith → distributed resilience |
| **Cattle, not pets** | Servers are disposable and identical, not carefully maintained unique instances. | Named application servers (prod-app-01) → anonymous container instances |
| **Immutable infrastructure** | Never patch a running server. Replace it with a new one built from a new image. | Manual server patches via SSH → container image replacement via rolling update |
| **Declarative configuration** | Describe the desired state, let the platform reconcile to achieve it. | Imperative deployment scripts → Kubernetes YAML / Terraform HCL |
| **Loose coupling** | Services interact through well-defined APIs, not shared databases or in-process calls. | Shared monolith DB with cross-module JOINs → service-owned data stores |

### Connecting to Your Experience

**A typical enterprise monolith** (large Java EE or Spring applications deployed as WAR/EAR files):
- Single deployable unit (WAR/EAR on Tomcat/WebSphere/JBoss)
- Shared relational database (all modules share tables)
- Vertical scaling (bigger server, more RAM)
- Long deployment cycles (monthly or quarterly releases)
- Tight coupling between modules through shared data models and libraries

**A modular monolith or service-oriented application** is a step toward cloud-native:
- Service-based module definitions
- Some external configuration
- Containerized deployment possible
- But still often deployed as a single unit with a shared database

**True cloud-native** goes further:
- Each bounded context is an independent service
- Each service owns its data store
- Services communicate via APIs and events
- Each service scales independently
- Each service has its own CI/CD pipeline

---

## 2. The 12-Factor App Methodology

Created by engineers at Heroku, the [12-Factor App](https://12factor.net/) defines best practices for building cloud-native applications. Every factor below includes the Java/Spring Boot implementation.

### Factor 1: Codebase — One codebase tracked in revision control, many deploys

```
✅ DO: One Git repo → builds one deployable service
❌ DON'T: Multiple services in one repo sharing code through source-level dependencies

Monorepo is acceptable IF each service has independent build/deploy pipeline.
```

**Spring Boot context:** One `@SpringBootApplication` class per service. One `pom.xml` or `build.gradle` per service (or a multi-module project with independent modules that each produce their own artifact).

### Factor 2: Dependencies — Explicitly declare and isolate dependencies

```xml
<!-- ✅ Good — everything declared explicitly in pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

```java
// ❌ Bad — relying on system tools not declared as dependencies
Runtime.getRuntime().exec("curl http://external-api.com/data");
// Don't assume curl is installed in the container
```

**Spring Boot context:** Spring Boot's "fat JAR" approach naturally isolates dependencies. Use `spring-boot-maven-plugin` to package everything into a self-contained JAR. Never rely on system-level libraries that aren't part of your container image.

### Factor 3: Config — Store config in the environment

This is one of the most commonly violated factors in enterprise Java.

```java
// ❌ VIOLATION — hardcoded credentials and URLs
@Service
public class PaymentService {
    private final String apiUrl = "https://payments.prod.example.com/api/v1";
    private final String apiKey = "sk_live_abc123def456";
    
    public PaymentResult processPayment(PaymentRequest request) {
        // Uses hardcoded production URL and key
        return restTemplate.postForObject(apiUrl, request, PaymentResult.class);
    }
}
```

```java
// ✅ FIX — externalized configuration
@Service
public class PaymentService {
    @Value("${payment.api.url}")
    private String apiUrl;

    @Value("${payment.api.key}")
    private String apiKey;

    public PaymentResult processPayment(PaymentRequest request) {
        return restTemplate.postForObject(apiUrl, request, PaymentResult.class);
    }
}
```

```yaml
# application.yml — uses environment variables
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

payment:
  api:
    url: ${PAYMENT_API_URL}
    key: ${PAYMENT_API_KEY}

server:
  port: ${PORT:8080}  # Default to 8080 if PORT not set
```

**In Kubernetes**, the API key belongs in a Secret, and the URL in a ConfigMap:

```yaml
# ConfigMap for non-sensitive config
apiVersion: v1
kind: ConfigMap
metadata:
  name: payment-config
data:
  PAYMENT_API_URL: "https://payments.prod.example.com/api/v1"

---
# Secret for sensitive config
apiVersion: v1
kind: Secret
metadata:
  name: payment-secrets
type: Opaque
stringData:
  PAYMENT_API_KEY: "sk_live_abc123def456"
```

### Factor 4: Backing services — Treat backing services as attached resources

```
Backing services are anything the app consumes over the network:
┌─────────────────────────────────────────────────────────┐
│  Your Application                                        │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │PostgreSQL│  │  Redis   │  │  Kafka   │  │ Twilio  ││
│  │ (DB)     │  │ (Cache)  │  │ (Queue)  │  │ (SMS)   ││
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘│
│       ↕              ↕             ↕            ↕       │
│  All attached via connection strings / URLs              │
│  Swappable without code changes                          │
└─────────────────────────────────────────────────────────┘

✅ Rule: You should be able to swap a local PostgreSQL for Amazon RDS
   by changing only the connection string (config), not the code.
```

**Spring Boot context:** Spring's abstraction layers (JPA, Spring Data, JMS) naturally support this. Just change the datasource URL in the environment.

### Factor 5: Build, release, run — Strictly separate build and run stages

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    BUILD     │ ──→ │   RELEASE   │ ──→ │     RUN     │
│              │     │             │     │             │
│ Source code  │     │ Build       │     │ Execute     │
│ + deps  →   │     │ artifact +  │     │ release in  │
│ build        │     │ config →   │     │ target env  │
│ artifact     │     │ release    │     │             │
│              │     │             │     │             │
│ mvn package  │     │ docker     │     │ kubectl     │
│              │     │ build      │     │ apply       │
└─────────────┘     └─────────────┘     └─────────────┘

✅ Each release is immutable and has a unique ID (e.g., image tag v1.2.3)
✅ You can roll back by deploying a previous release
❌ Never modify code or config at runtime (no SSH to patch production)
```

**Spring Boot pipeline:**
- **Build:** `mvn clean package -DskipTests=false` → produces `app.jar`
- **Release:** `docker build -t myapp:v1.2.3 .` → produces immutable image with config baked in
- **Run:** `kubectl apply -f deployment.yaml` or `docker run myapp:v1.2.3`

### Factor 6: Processes — Execute the app as one or more stateless processes

```java
// ❌ ANTI-PATTERN — in-memory session state
@RestController
public class CartController {
    @PostMapping("/cart/add")
    public ResponseEntity<Cart> addToCart(HttpServletRequest request, @RequestBody Item item) {
        HttpSession session = request.getSession();
        Cart cart = (Cart) session.getAttribute("cart");  // Stored in JVM heap
        if (cart == null) cart = new Cart();
        cart.add(item);
        session.setAttribute("cart", cart);  // Lost when pod restarts or another pod handles next request
        return ResponseEntity.ok(cart);
    }
}
```

```java
// ✅ FIX — external session store with Spring Session + Redis
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class SessionConfig {
    // Sessions are now stored in Redis
    // Any pod can serve any request
    // Sessions survive pod restarts and scaling events
}
```

```yaml
# application.yml
spring:
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
  session:
    store-type: redis
```

**Enterprise monolith context:** Traditional Java application servers often store session state in the JVM's heap memory, sometimes with "sticky sessions" on the load balancer. Cloud-native requires externalizing state to Redis or a similar store, so any instance can serve any request.

### Factor 7: Port binding — Export services via port binding

```
✅ The app is self-contained and binds to a port
✅ No external web server required (no separate Tomcat installation)

Spring Boot does this natively:
- Embedded Tomcat/Netty/Undertow
- Binds to PORT environment variable or default 8080
- The app IS the web server
```

### Factor 8: Concurrency — Scale out via the process model

```
┌───────────────────────────────────────────────────┐
│                 Load Balancer                       │
└────────┬──────────┬──────────┬────────────────────┘
         │          │          │
    ┌────┴──┐  ┌───┴───┐  ┌──┴────┐
    │ Web   │  │ Web   │  │ Web   │    ← HTTP request handling
    │ Pod 1 │  │ Pod 2 │  │ Pod 3 │       (HPA scales these)
    └───────┘  └───────┘  └───────┘

    ┌────────┐  ┌────────┐
    │Worker 1│  │Worker 2│    ← Background job processing
    └────────┘  └────────┘       (separate deployment, scales independently)

✅ Scale by adding more processes/pods, not bigger JVMs
✅ Different process types for different workloads (web, worker, scheduler)
❌ Don't scale by adding threads inside a single giant JVM
```

### Factor 9: Disposability — Maximize robustness with fast startup and graceful shutdown

```java
@Component
public class GracefulShutdownHandler {

    private static final Logger log = LoggerFactory.getLogger(GracefulShutdownHandler.class);
    
    private final KafkaListenerEndpointRegistry kafkaRegistry;
    
    public GracefulShutdownHandler(KafkaListenerEndpointRegistry kafkaRegistry) {
        this.kafkaRegistry = kafkaRegistry;
    }

    @PreDestroy
    public void onShutdown() {
        log.info("Shutdown signal received. Starting graceful shutdown...");
        
        // 1. Stop accepting new Kafka messages
        kafkaRegistry.stop();
        log.info("Kafka listeners stopped.");
        
        // 2. In-flight HTTP requests are handled by Spring's graceful shutdown
        // 3. Database connections are closed by the connection pool
        // 4. Flush remaining logs
        log.info("Graceful shutdown complete.");
    }
}
```

```yaml
# application.yml — graceful shutdown configuration
server:
  shutdown: graceful           # Wait for in-flight requests to complete

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # Max wait time before force kill
```

**Startup optimization tips:**
- Avoid heavy initialization in constructors
- Use lazy initialization where possible: `spring.main.lazy-initialization=true`
- Consider GraalVM native images for sub-second startup (Spring Boot 3+ supports this)
- Set appropriate JVM flags: `-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0`

### Factor 10: Dev/prod parity — Keep development, staging, and production as similar as possible

```yaml
# docker-compose.yml — local development environment matching production
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=local
      - DATABASE_URL=jdbc:postgresql://db:5432/myapp
      - REDIS_HOST=redis
      - KAFKA_BOOTSTRAP_SERVERS=kafka:29092
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
      kafka:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: devpassword
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://kafka:29092,CONTROLLER://kafka:29093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:29093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qk"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    ports:
      - "29092:29092"
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "kafka:29092"]
      interval: 10s
      timeout: 10s
      retries: 5
```

```
✅ Same database type in dev and prod (PostgreSQL everywhere, not H2 in dev)
✅ Same message broker (Kafka everywhere, not in-memory in dev)
✅ Same cache (Redis everywhere)
❌ Don't use H2 in dev and PostgreSQL in production — SQL dialects differ
❌ Don't use in-memory queues in dev and Kafka in production
```

### Factor 11: Logs — Treat logs as event streams

```xml
<!-- logback-spring.xml — structured JSON logging to stdout -->
<configuration>
    <springProperty scope="context" name="SERVICE_NAME" source="spring.application.name" 
                    defaultValue="unknown-service"/>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"service":"${SERVICE_NAME}"}</customFields>
            <fieldNames>
                <timestamp>timestamp</timestamp>
                <version>[ignore]</version>
            </fieldNames>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

Output example:
```json
{
  "timestamp": "2026-03-15T10:30:45.123Z",
  "level": "INFO",
  "logger_name": "com.example.OrderService",
  "message": "Order created successfully",
  "service": "order-service",
  "traceId": "abc123def456",
  "spanId": "789ghi",
  "orderId": "ORD-001",
  "customerId": "CUST-042"
}
```

```
✅ Write to stdout/stderr (not log files)
✅ Use JSON format for machine-parseable structured logging
✅ Include traceId/spanId for distributed tracing correlation
✅ Let the platform (Fluentd, CloudWatch, ELK) collect, aggregate, and route logs
❌ Don't write to /var/log/app.log (who rotates it? who ships it?)
❌ Don't use plain-text logs in production (hard to search and aggregate)
```

### Factor 12: Admin processes — Run admin/management tasks as one-off processes

```yaml
# Database migration as a Kubernetes Job (not on app startup)
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration-v1-2-3
spec:
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: migration
        image: myapp:v1.2.3     # Same image as the application
        command: ["java", "-jar", "app.jar", 
                  "--spring.flyway.enabled=true",
                  "--spring.main.web-application-type=none"]
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
      restartPolicy: Never
```

```
✅ Database migrations run as Kubernetes Jobs, not on app startup
✅ Data fixes run as one-off pods with the same codebase/image
✅ Use the same environment and config as the running application
❌ Don't SSH into a pod to run scripts
❌ Don't run migrations automatically on every app startup (race conditions with multiple pods)
```

---

## 3. Beyond 12-Factor: The 15-Factor App

Three additional factors for modern cloud-native applications:

### Factor 13: API First

Design the API before writing implementation code. Use contract-first development.

```
1. Write the OpenAPI specification (YAML/JSON)
2. Review with consumers (frontend team, mobile team, partner)
3. Generate server stubs and client SDKs
4. Implement the server
5. Verify with contract tests

Benefits:
- Frontend and backend can develop in parallel
- API contract is the source of truth
- Reduces integration surprises
- Enables consumer-driven contract testing (Pact)
```

### Factor 14: Telemetry

Build observability into every service from day one.

```
Three types of telemetry:

1. APPLICATION PERFORMANCE
   - Request latency (p50, p95, p99)
   - Error rates
   - Throughput (requests/sec)
   → Tools: Micrometer + Prometheus + Grafana

2. DOMAIN-SPECIFIC / BUSINESS METRICS
   - Orders per minute
   - Payment success rate
   - Cart abandonment rate
   → Tools: Custom Micrometer counters/gauges

3. HEALTH AND SYSTEM METRICS
   - JVM heap usage, GC pauses
   - Pod CPU/memory utilization
   - Database connection pool stats
   → Tools: Spring Boot Actuator + Prometheus
```

### Factor 15: Authentication & Authorization

Security is not optional. Every service must handle auth properly.

```
Principles:
- Token-based authentication (JWT via OAuth 2.0 / OpenID Connect)
- NEVER build your own auth system — use identity providers (Keycloak, Auth0, Okta)
- Role-Based Access Control (RBAC) or Attribute-Based Access Control (ABAC)
- Zero-trust: verify every request, even between internal services
- Use mTLS for service-to-service communication (automated by service mesh)
```

---

## 4. Cloud-Native Maturity Model

Where does your application sit? Use this model to assess current state and plan migration.

```
Level 0: CLOUD HOSTILE
├── Hardcoded IPs and file paths
├── Manual deployment (copy JAR to server, restart)
├── State stored on local filesystem
├── No health checks, no monitoring
└── Example: Legacy Java EE app on a physical server
    Effort to next level: LOW — externalize config, containerize

Level 1: CLOUD READY
├── Externalized configuration (environment variables or config files)
├── Uses managed services (cloud DB, cloud cache)
├── Containerized (Docker) but still a monolith
├── Basic CI/CD pipeline
└── Example: Spring Boot monolith on EC2 or Cloud Foundry
    Effort to next level: MEDIUM — add 12-factor compliance, health checks

Level 2: CLOUD FRIENDLY
├── Follows most 12-factor principles
├── Horizontal scaling works (stateless processes)
├── CI/CD pipeline with automated testing
├── Health checks and basic monitoring
├── Structured logging to stdout
└── Example: Well-architected Spring Boot app on Kubernetes
    Effort to next level: HIGH — decompose into microservices

Level 3: CLOUD NATIVE
├── Microservices architecture with bounded contexts
├── Event-driven communication (Kafka, async messaging)
├── Full observability (metrics, traces, structured logs)
├── Infrastructure as Code (Terraform, Helm)
├── Auto-scaling based on metrics (HPA, KEDA)
├── Resilience patterns (circuit breakers, retries, bulkheads)
└── Example: Purpose-built microservices on Kubernetes
    Effort to next level: MEDIUM — add serverless where appropriate

Level 4: CLOUD NATIVE + SERVERLESS
├── Functions-as-a-Service for appropriate workloads
├── Event-driven at the core (event sourcing, CQRS)
├── Pay-per-use economics (no idle resources)
├── Minimal operational overhead
└── Example: Mix of Kubernetes services + AWS Lambda/Azure Functions
```

---

## 5. The CNCF Landscape

The [CNCF landscape](https://landscape.cncf.io/) contains 1000+ projects. Here is a curated map of the most important categories and the graduated/key projects you should know:

```
┌────────────────────────────────────────────────────────────────┐
│                     CNCF LANDSCAPE (Curated)                    │
├────────────────────┬───────────────────────────────────────────┤
│ ORCHESTRATION      │ Kubernetes ★                               │
├────────────────────┼───────────────────────────────────────────┤
│ CONTAINER RUNTIME  │ containerd ★, CRI-O, gVisor                │
├────────────────────┼───────────────────────────────────────────┤
│ SERVICE PROXY      │ Envoy ★, NGINX                             │
├────────────────────┼───────────────────────────────────────────┤
│ SERVICE MESH       │ Istio, Linkerd ★, Cilium                   │
├────────────────────┼───────────────────────────────────────────┤
│ NETWORKING         │ Cilium, CoreDNS ★, CNI                     │
├────────────────────┼───────────────────────────────────────────┤
│ STORAGE            │ etcd ★, Rook ★, Longhorn, MinIO            │
├────────────────────┼───────────────────────────────────────────┤
│ OBSERVABILITY      │ Prometheus ★, Grafana, Jaeger ★,           │
│                    │ OpenTelemetry ★, Fluentd ★                  │
├────────────────────┼───────────────────────────────────────────┤
│ CI/CD              │ Argo ★ (CD/Workflows/Rollouts),            │
│                    │ Flux ★, Tekton                              │
├────────────────────┼───────────────────────────────────────────┤
│ SECURITY           │ OPA ★, Falco ★, cert-manager, SPIFFE/SPIRE│
├────────────────────┼───────────────────────────────────────────┤
│ MESSAGING / API    │ NATS, CloudEvents, gRPC                    │
├────────────────────┼───────────────────────────────────────────┤
│ PACKAGE MGMT       │ Helm ★                                     │
└────────────────────┴───────────────────────────────────────────┘
                    ★ = CNCF Graduated project (production-ready)
```

**How to navigate the landscape:**
1. Start with graduated projects — they are production-proven and widely adopted
2. Don't try to learn everything — focus on what your architecture needs
3. Understand the categories, not every project within them
4. For Java developers: Kubernetes + Helm + Prometheus + ArgoCD is the core stack

---

## 6. Evolution: Monolith → SOA → Microservices → Serverless

### The Architecture Evolution Tree

```
MONOLITH (1990s-2000s)
│  Everything in one deployable unit
│  One database, one team, one release cycle
│  Example: Large Spring/Hibernate WAR on Tomcat
│
├──→ SOA - Service Oriented Architecture (2000s-2010s)
│    │  Enterprise Service Bus (ESB) as central hub
│    │  SOAP/WSDL web services
│    │  Shared canonical data model
│    │  Centralized governance and standards
│    │  Example: IBM WebSphere ESB, Oracle SOA Suite, MuleSoft
│    │
│    ├──→ MICROSERVICES (2010s-present)
│    │    │  Small, independently deployable services
│    │    │  Decentralized governance ("you build it, you run it")
│    │    │  Each service owns its data
│    │    │  Smart endpoints, dumb pipes (REST, gRPC, events)
│    │    │  Example: Netflix, Spotify, most modern SaaS platforms
│    │    │
│    │    ├──→ SERVERLESS / FaaS (2015s-present)
│    │    │    Event-triggered functions
│    │    │    No server management
│    │    │    Pay per execution (zero cost when idle)
│    │    │    Example: AWS Lambda, Azure Functions, Google Cloud Functions
│    │    │
│    │    └──→ SERVICE MESH (2017s-present)
│    │         Infrastructure layer for service-to-service communication
│    │         Sidecar proxy pattern (Envoy)
│    │         mTLS, observability, traffic management
│    │         Example: Istio, Linkerd, Cilium
│    │
│    └──→ MODULAR MONOLITH (modern alternative)
│         Well-structured monolith with module boundaries
│         Independent modules with enforced interfaces
│         Stepping stone to microservices if/when needed
│         Example: Shopify (modular Rails monolith)
```

### SOA vs. Microservices — Key Differences

| Aspect | SOA | Microservices |
|--------|-----|---------------|
| **Communication** | ESB (smart pipes) — routing, transformation, orchestration in middleware | REST/gRPC/events (dumb pipes) — "smart endpoints, dumb pipes" |
| **Data** | Shared database or canonical data model | Database per service — each service owns its data |
| **Governance** | Centralized — shared schemas, enterprise-wide standards bodies | Decentralized — each team chooses tech stack and data model |
| **Service size** | Large services covering broad business domains | Small, focused services with single responsibility |
| **Deployment** | Often bundled or coordinated releases | Independent deployment per service |
| **Protocol** | SOAP/XML heavy, WS-* standards | REST/JSON, gRPC/Protobuf, async events |
| **Reuse strategy** | Via shared services and canonical models | Via APIs — prefer duplication over tight coupling |
| **Team structure** | Organized by technology layer (UI team, middleware team, DB team) | Organized by business capability (order team, payment team) |

---

## 7. Cloud-Native vs. Cloud-Hosted vs. Cloud-Ready

### Clear Definitions

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  CLOUD-HOSTED (Lift & Shift)                                   │
│  ├── Same application, running on cloud VMs                    │
│  ├── Minimal or zero changes to code                           │
│  ├── Still a monolith, still vertical scaling                  │
│  ├── Benefits: CapEx → OpEx, data center exit                  │
│  └── Example: Java EE monolith on AWS EC2 instances            │
│                                                                │
│  CLOUD-READY (Lift, Shift & Optimize)                          │
│  ├── Containerized application (Docker)                        │
│  ├── Uses some managed services (RDS, ElastiCache)             │
│  ├── Externalized configuration                                │
│  ├── Auto-scaling configured                                   │
│  ├── Benefits: better resource utilization, managed operations  │
│  └── Example: Containerized Spring Boot monolith on Kubernetes │
│                                                                │
│  CLOUD-NATIVE (Re-architect)                                   │
│  ├── Microservices architecture with bounded contexts           │
│  ├── Event-driven communication (Kafka, async messaging)       │
│  ├── Full observability (metrics, traces, structured logs)     │
│  ├── CI/CD with GitOps, Infrastructure as Code                 │
│  ├── Resilience patterns (circuit breakers, retries)           │
│  ├── Benefits: max agility, resilience, independent scaling    │
│  └── Example: Purpose-built microservices on Kubernetes        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Migration Strategy Decision Matrix

| Factor | Cloud-Hosted (Lift & Shift) | Cloud-Ready (Optimize) | Cloud-Native (Re-architect) |
|--------|---------------------------|----------------------|---------------------------|
| **Timeline** | Weeks | Months | Quarters to years |
| **Cost (short-term)** | Low | Medium | High |
| **Cost (long-term)** | High (over-provisioned VMs) | Medium | Low (efficient resource use) |
| **Risk** | Low | Medium | High (organizational change) |
| **Agility gain** | Minimal | Moderate | Maximum |
| **Operational effort** | High (still managing VMs) | Medium (some managed services) | Low (platform handles operations) |
| **When to choose** | Data center exit deadline, quick migration | Good enough for now, incremental path | Strategic investment, high change rate, scaling needs |

---

## 8. Exercises

### Exercise 1: 12-Factor Audit

Take a Java application you have worked on (a Spring Boot monolith or enterprise application) and audit it against all 12 factors.

For each factor:
1. Rate compliance: ✅ Compliant / 🟡 Partially / ❌ Non-compliant
2. Describe the current state
3. Describe what would need to change for compliance
4. Estimate the effort: Low / Medium / High

**Deliverable:** A 12-factor compliance table with remediation plan.

| Factor | Status | Current State | Remediation | Effort |
|--------|--------|--------------|-------------|--------|
| 1. Codebase | | | | |
| 2. Dependencies | | | | |
| ... | | | | |
| 12. Admin processes | | | | |

---

### Exercise 2: Cloud-Native Maturity Assessment

Using the maturity model from Section 4, assess two systems:
1. A traditional enterprise monolith deployed on-premise or on VMs
2. A containerized Spring Boot application with CI/CD pipeline

For each system, document:
- Current maturity level (0-4)
- Which criteria at the current level are met
- Which criteria at the next level are not met
- Specific changes needed to advance one level
- Estimated effort and risk for advancement

**Deliverable:** A maturity comparison document with gap analysis.

---

### Exercise 3: Architecture Decision — When NOT to Go Cloud-Native

Design a decision framework for when cloud-native is NOT the right approach. Consider scenarios:
- Small team (2-3 developers)
- Application with <100 users
- Regulatory requirements for data sovereignty
- Application with no need for elastic scaling
- Tight budget and timeline
- Simple CRUD application with minimal business logic

**Deliverable:** A decision tree or flowchart with justification for each branch.

---

### Exercise 4: Hands-On — 12-Factor Spring Boot App

Create a Spring Boot application that demonstrably follows all 12 factors:

1. Create a simple REST API (e.g., a task manager with CRUD operations)
2. Externalize all configuration via environment variables
3. Add structured JSON logging to stdout (logback-spring.xml with JSON encoder)
4. Add health check endpoints (`/actuator/health`, `/actuator/info`)
5. Create a `Dockerfile` with multi-stage build (JDK for build, JRE for runtime)
6. Create a `docker-compose.yml` with PostgreSQL and Redis
7. Implement graceful shutdown (`server.shutdown=graceful`)
8. Use Flyway for database migrations (run as separate step, not on startup)
9. Add Prometheus metrics endpoint (`/actuator/prometheus`)
10. Document which 12-factor principle each design decision satisfies

**Deliverable:** A working GitHub repository with README documenting 12-factor compliance.

---

## 9. Self-Check Questions

Answer these questions without looking at the notes. Then check your answers in Section 10.

1. What are the four pillars of cloud-native architecture?

2. Explain the difference between "cloud-native" and "cloud-hosted." Give an example of each.

3. Which 12-Factor principle does this code violate? How would you fix it?
   ```java
   @Service
   public class PaymentService {
       private final String apiUrl = "https://payments.prod.example.com/api/v1";
       private final String apiKey = "sk_live_abc123";
   }
   ```

4. Why should cloud-native applications write logs to stdout instead of files?

5. A team has a Spring Boot application that stores user session data in an HTTP session (in-memory). They want to scale to 5 instances behind a load balancer. What problem will they face? What are two possible solutions?

6. What is the difference between SOA and microservices? Name three key differences.

7. Your CTO asks: "We have a monolithic Java application. Should we rewrite it as microservices?" What questions would you ask before giving an answer?

8. What are the three additional factors in the 15-factor app methodology?

9. Explain the "cattle, not pets" principle. How does it relate to immutable infrastructure?

10. What level of the cloud-native maturity model would you assign to: (a) a VM-based enterprise monolith with hardcoded config, (b) a containerized Spring Boot app with CI/CD and health checks, (c) a serverless event-driven architecture on AWS Lambda?

---

## 10. Answers

### Answer 1
The four pillars are:
1. **Microservices** — small, independent, focused services with single responsibility
2. **Containers** — lightweight, portable runtime environments (Docker, OCI)
3. **CI/CD** — automated build, test, and deployment pipelines (continuous integration and delivery)
4. **DevOps** — culture of shared ownership between development and operations ("you build it, you run it")

### Answer 2
- **Cloud-hosted:** A traditional monolithic application running on cloud VMs with minimal code changes. Example: An enterprise Java monolith deployed on AWS EC2 instances — same WAR file, same architecture, just different infrastructure. The key characteristic: you changed *where* it runs.
- **Cloud-native:** An application designed from the ground up to leverage cloud capabilities — microservices, containers, event-driven, auto-scaling. Example: A microservices-based e-commerce platform on Kubernetes with each service independently deployable and scalable. The key characteristic: you changed *how* it's built.

The key difference: cloud-hosted is about **where** it runs, cloud-native is about **how** it's designed and built.

### Answer 3
This violates **Factor 3: Config** — store config in the environment.

**Problems:**
- Production URL is hardcoded — changing it requires a code change, build, and redeployment
- API key (a secret!) is in source code — anyone with repo access can see it
- The same code cannot be used in dev, staging, and production without modification

**Fix:**
```java
@Service
public class PaymentService {
    @Value("${payment.api.url}")
    private String apiUrl;

    @Value("${payment.api.key}")
    private String apiKey;
}
```
With `application.yml`:
```yaml
payment:
  api:
    url: ${PAYMENT_API_URL}
    key: ${PAYMENT_API_KEY}
```
In Kubernetes, the URL goes in a ConfigMap and the API key in a Secret.

### Answer 4
Reasons to write logs to stdout:
1. **Separation of concerns** — the application generates logs, the platform (Kubernetes, CloudWatch, ELK) handles collection, aggregation, storage, and routing
2. **Portability** — the same application works in Docker, Kubernetes, bare metal, or any cloud without changing log configuration
3. **No disk management** — no log rotation, no disk space monitoring, no file permissions
4. **Real-time streaming** — stdout is captured and forwarded in real-time to centralized logging systems (Fluentd, Filebeat)
5. **Container-friendly** — `docker logs` and `kubectl logs` read from stdout/stderr natively

### Answer 5
**Problem:** With round-robin load balancing (default), requests from the same user hit different instances. Each instance has its own in-memory session store, so session data (cart, login state) is lost or inconsistent when the next request goes to a different pod.

**Solution 1: External session store** — Spring Session with Redis:
```java
@EnableRedisHttpSession
public class SessionConfig { }
```
All instances share the same session store in Redis. Any pod can serve any request.

**Solution 2: Stateless design** — Store no session on the server. Use JWT tokens that contain the necessary state. The client sends the token with every request, so any instance can handle it.

**Why not sticky sessions?** Sticky sessions (load balancer affinity) create uneven load distribution and if the "sticky" instance goes down, all its sessions are lost. It defeats the purpose of horizontal scaling.

### Answer 6

| Aspect | SOA | Microservices |
|--------|-----|---------------|
| **Communication** | ESB (Enterprise Service Bus) — smart routing, transformation, orchestration in middleware | Direct service-to-service via REST/gRPC or async messaging — "smart endpoints, dumb pipes" |
| **Data** | Shared database or canonical data model | Database per service — each service owns its data |
| **Governance** | Centralized — shared schemas, enterprise standards | Decentralized — each team chooses its own tech stack and data model |

Additional differences: SOA services are large; microservices are small and focused. SOA uses SOAP/XML; microservices prefer REST/JSON or gRPC. SOA emphasizes reuse through shared services; microservices prefer duplication over coupling.

### Answer 7
Questions to ask before recommending microservices:
1. **Why?** What specific problem are you trying to solve? Slow deployments? Scaling bottleneck? Team autonomy?
2. **Team size?** Do you have enough teams (3+ teams of 5-8 people) to own separate services? Microservices without team ownership becomes a distributed monolith.
3. **Domain maturity?** Do you understand the domain boundaries well? Premature decomposition leads to wrong service boundaries, which are expensive to fix.
4. **Operational readiness?** Do you have CI/CD, monitoring, distributed tracing, container orchestration? Microservices without operational maturity is chaos.
5. **Can you start incrementally?** Can you use the strangler fig pattern to extract services gradually rather than a big-bang rewrite?
6. **Deployment frequency?** If you deploy once a month, microservices add complexity without much benefit.
7. **Distributed systems comfort?** Is the team ready for distributed transactions, eventual consistency, network partition handling?

### Answer 8
The three additional factors (15-factor app):
1. **API First** — design APIs before implementation, use contract-first approach (OpenAPI spec first, then code)
2. **Telemetry** — built-in monitoring, metrics, health checks, distributed tracing (observability from day one)
3. **Authentication & Authorization** — token-based auth (OAuth 2.0 / JWT), never build your own auth, zero-trust (verify every request)

### Answer 9
**Cattle, not pets:**
- **Pets:** Servers have names (prod-server-01), are cared for individually, get patched and nursed back to health when sick. If one dies, everyone panics and works overtime to fix it.
- **Cattle:** Servers have numbers (pod-4a7f-x9z2), are identical and interchangeable. If one is sick, you kill it and replace it instantly. Nobody cares about individual instances.

**Relationship to immutable infrastructure:**
Immutable infrastructure means you never modify a running server. Instead:
1. Build a new container image with the fix/update
2. Deploy new instances with the new image (rolling update)
3. Old instances are terminated

This only works if servers are "cattle" — disposable and replaceable. If you have "pets" with unique snowflake configurations, you can't simply replace them. Immutable infrastructure *requires* the cattle mindset.

### Answer 10
- **(a) VM-based enterprise monolith with hardcoded config:** **Level 0 (Cloud Hostile)** — Hardcoded configuration violates 12-factor. Likely manual deployment, no health checks, no containerization.

- **(b) Containerized Spring Boot app with CI/CD and health checks:** **Level 2 (Cloud Friendly)** — Containerized ✅, CI/CD ✅, health checks ✅, likely follows most 12-factor principles. But probably still a single service (not microservices), may lack full observability, event-driven communication, and auto-scaling.

- **(c) Serverless event-driven architecture on AWS Lambda:** **Level 4 (Cloud Native + Serverless)** — Event-driven ✅, Functions-as-a-Service ✅, pay-per-execution ✅, minimal operational overhead ✅. This is the highest maturity level.

---

## Key Takeaways

1. **Cloud-native is a mindset**, not a technology. It's about designing for the cloud's strengths: elasticity, managed services, and automation.
2. **The 12-Factor methodology is your checklist** — audit every application against it. Most enterprise apps violate at least 4-5 factors.
3. **Not everything needs to be cloud-native.** The maturity model helps you choose the right level for each workload. Level 2 (Cloud Friendly) is often sufficient.
4. **Your enterprise experience is valuable.** The anti-patterns you've seen in monoliths are the motivation for cloud-native principles. Pain → principle → practice.
5. **Start with the strangler fig pattern** — don't do big-bang rewrites. Extract services incrementally.

---

## References

- [The Twelve-Factor App](https://12factor.net/) — Original methodology
- [CNCF Cloud Native Interactive Landscape](https://landscape.cncf.io/) — Project ecosystem
- [Microsoft Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/) — Architecture styles and patterns
- [Martin Fowler — Microservices](https://martinfowler.com/articles/microservices.html) — Original microservices article
- "Fundamentals of Software Architecture" — Mark Richards & Neal Ford

---

*Next lesson: [02 — Architecture Styles & Patterns](02-architecture-styles.md)*