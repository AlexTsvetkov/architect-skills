# Lesson 01: Cloud-Native Architecture Fundamentals

> **Prerequisites:** Java development experience, basic understanding of web applications  
> **Time estimate:** 4-6 hours  
> **Parallel reading:** "Fundamentals of Software Architecture" — Richards & Ford (Part I)

---

## Table of Contents

1. [What is Cloud-Native?](#1-what-is-cloud-native)
2. [The 12-Factor App Methodology](#2-the-12-factor-app-methodology)
3. [Beyond 12-Factor: The 15-Factor App](#3-beyond-12-factor)
4. [Cloud-Native Maturity Model](#4-cloud-native-maturity-model)
5. [The CNCF Landscape](#5-the-cncf-landscape)
6. [Evolution: Monolith → SOA → Microservices → Serverless](#6-evolution)
7. [Cloud-Native vs. Cloud-Hosted vs. Cloud-Ready](#7-cloud-native-vs-cloud-hosted)
8. [Exercises](#8-exercises)
9. [Self-Check Questions](#9-self-check-questions)
10. [Answers](#10-answers)

---

## 1. What is Cloud-Native?

### The CNCF Definition

The Cloud Native Computing Foundation (CNCF) defines cloud-native as:

> *"Cloud-native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach."*

### What This Really Means

Cloud-native is **not** just "running your app in the cloud." It's a set of architectural principles and practices that:

1. **Embrace the cloud's elasticity** — design for horizontal scaling, not bigger servers
2. **Assume failure** — every component can and will fail; design for resilience
3. **Automate everything** — from deployment to recovery to scaling
4. **Optimize for speed of change** — small, frequent, safe deployments

### The Four Pillars of Cloud-Native

```
┌─────────────────────────────────────────────────────┐
│                  CLOUD-NATIVE                        │
├─────────────┬──────────────┬───────────┬────────────┤
│ Microservices│  Containers  │   CI/CD   │  DevOps    │
│             │              │           │            │
│ Small,      │ Lightweight, │ Automated │ Culture of │
│ independent │ portable     │ build,    │ shared     │
│ services    │ runtime      │ test,     │ ownership  │
│             │ environment  │ deploy    │            │
└─────────────┴──────────────┴───────────┴────────────┘
```

### Key Principles

| Principle | Description | Your SAP Commerce Context |
|-----------|-------------|--------------------------|
| **Design for failure** | Assume any component can crash. Build in retries, circuit breakers, fallbacks. | Hybris single-point-of-failure → distributed resilience |
| **Cattle, not pets** | Servers are disposable and identical, not carefully maintained unique instances. | Named Hybris servers → anonymous container instances |
| **Immutable infrastructure** | Never patch a running server. Replace it with a new one. | Manual Hybris patches → container image replacement |
| **Declarative configuration** | Describe the desired state, let the platform reconcile. | Imperative scripts → Kubernetes YAML / Terraform |
| **Loose coupling** | Services interact through well-defined APIs, not shared databases. | Hybris shared DB → service-owned data stores |

### Connecting to Your Experience

**SAP Commerce (Hybris)** is a classic enterprise monolith:
- Single deployable unit (WAR/EAR)
- Shared relational database (all extensions share tables)
- Vertical scaling (bigger server)
- Long deployment cycles
- Tight coupling between modules through the type system

**SAP CAP** is a step toward cloud-native:
- Service-based model definitions
- Multi-tenancy support
- Cloud Foundry / Kubernetes deployment
- But still often deployed as a single service

**True cloud-native** goes further:
- Each bounded context is an independent service
- Each service owns its data
- Services communicate via APIs and events
- Each service scales independently
- Each service has its own CI/CD pipeline

---

## 2. The 12-Factor App Methodology

Created by engineers at Heroku, the [12-Factor App](https://12factor.net/) defines best practices for building cloud-native applications. Let's go through each with Java/Spring Boot context.

### Factor 1: Codebase — One codebase tracked in revision control, many deploys

```
✅ DO: One Git repo → builds one deployable service
❌ DON'T: Multiple services in one repo sharing code through source-level dependencies

Monorepo is acceptable IF each service has independent build/deploy pipeline
```

**Spring Boot context:** One `@SpringBootApplication` class per service. One `pom.xml` or `build.gradle` per service (or a multi-module project with independent modules).

### Factor 2: Dependencies — Explicitly declare and isolate dependencies

```
✅ DO: Declare ALL dependencies in pom.xml/build.gradle
❌ DON'T: Rely on system-installed libraries or shared classpath

# Good - everything declared
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

# Bad - relying on system tools
Runtime.getRuntime().exec("curl ...") // Don't assume curl is installed
```

**Spring Boot context:** Spring Boot's "fat JAR" approach naturally isolates dependencies. Use `spring-boot-maven-plugin` to package everything.

### Factor 3: Config — Store config in the environment

```
✅ DO: Externalize ALL environment-specific config
❌ DON'T: Hardcode URLs, credentials, or feature flags

# application.yml — uses environment variables
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

server:
  port: ${PORT:8080}  # Default to 8080 if PORT not set
```

**Spring Boot context:** Use Spring profiles (`application-dev.yml`, `application-prod.yml`), environment variables, or Spring Cloud Config Server. In Kubernetes, use ConfigMaps and Secrets.

### Factor 4: Backing services — Treat backing services as attached resources

```
Backing services are anything the app consumes over the network:
- Databases (PostgreSQL, MySQL)
- Message queues (RabbitMQ, Kafka)
- Caches (Redis)
- SMTP services
- Third-party APIs (Twilio, Stripe)

✅ Rule: You should be able to swap a local MySQL for Amazon RDS
by changing only the connection string (config), not the code.
```

**Spring Boot context:** Spring's abstraction layers (JPA, JMS, Spring Data) naturally support this. Just change the datasource URL.

### Factor 5: Build, release, run — Strictly separate build and run stages

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  BUILD  │ ──→ │ RELEASE │ ──→ │   RUN   │
│         │     │         │     │         │
│ Code +  │     │ Build + │     │ Execute │
│ deps →  │     │ config →│     │ release │
│ build   │     │ release │     │ in env  │
│ artifact│     │         │     │         │
└─────────┘     └─────────┘     └─────────┘

✅ Each release is immutable and has a unique ID
✅ You can roll back by deploying a previous release
❌ Never modify code at runtime
```

**Spring Boot context:**
- **Build:** `mvn clean package` → produces `app.jar`
- **Release:** `docker build` → produces `app:v1.2.3` image with config
- **Run:** `kubectl apply` or `docker run`

### Factor 6: Processes — Execute the app as one or more stateless processes

```
✅ DO: Store all state in backing services (DB, Redis, S3)
❌ DON'T: Store state in local memory or filesystem expecting it to persist

# Bad - in-memory session
HttpSession session = request.getSession();
session.setAttribute("cart", cart);  // Lost when pod restarts

# Good - external session store
@EnableRedisHttpSession
public class SessionConfig {
    // Sessions stored in Redis, survive pod restarts/scaling
}
```

**Your SAP Commerce context:** Hybris often stores session state in the JVM. Cloud-native requires externalizing this to Redis or a similar store.

### Factor 7: Port binding — Export services via port binding

```
✅ The app is self-contained and binds to a port
✅ No external web server required (no Tomcat installation)

Spring Boot does this natively:
- Embedded Tomcat/Netty
- Binds to PORT environment variable
```

### Factor 8: Concurrency — Scale out via the process model

```
┌─────────────────────────────────────────────┐
│              Load Balancer                    │
└──────┬──────────┬──────────┬────────────────┘
       │          │          │
  ┌────┴──┐  ┌───┴───┐  ┌──┴────┐
  │ Web   │  │ Web   │  │ Web   │    ← HTTP request handling
  │ Pod 1 │  │ Pod 2 │  │ Pod 3 │
  └───────┘  └───────┘  └───────┘

  ┌────────┐  ┌────────┐
  │Worker 1│  │Worker 2│    ← Background job processing
  └────────┘  └────────┘

✅ Scale by adding more processes/pods, not bigger JVMs
✅ Different process types for different workloads
```

### Factor 9: Disposability — Maximize robustness with fast startup and graceful shutdown

```java
// Graceful shutdown in Spring Boot
@PreDestroy
public void onShutdown() {
    // 1. Stop accepting new requests
    // 2. Finish in-flight requests
    // 3. Close database connections
    // 4. Flush logs
    log.info("Shutting down gracefully...");
}

// application.yml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

**Startup optimization:**
- Avoid heavy initialization in constructors
- Use lazy initialization where possible: `spring.main.lazy-initialization=true`
- Consider GraalVM native images for sub-second startup

### Factor 10: Dev/prod parity — Keep development, staging, and production as similar as possible

```
✅ Use Docker/Docker Compose for local dev
✅ Same database type in dev and prod (not H2 in dev, PostgreSQL in prod)
✅ Same messaging system (not in-memory in dev, Kafka in prod)

# docker-compose.yml for local development
services:
  app:
    build: .
    ports: ["8080:8080"]
    environment:
      - DATABASE_URL=jdbc:postgresql://db:5432/myapp
  db:
    image: postgres:16
    environment:
      - POSTGRES_DB=myapp
  redis:
    image: redis:7
  kafka:
    image: confluentinc/cp-kafka:7.5.0
```

### Factor 11: Logs — Treat logs as event streams

```
✅ Write to stdout/stderr (not log files)
✅ Let the platform aggregate and route logs

# logback-spring.xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <!-- JSON format for structured logging -->
      <pattern>{"timestamp":"%d","level":"%level","service":"${SERVICE_NAME}",
        "traceId":"%X{traceId}","message":"%msg"}%n</pattern>
    </encoder>
  </appender>
</configuration>

# The platform (ELK, Fluentd, CloudWatch) collects from stdout
```

### Factor 12: Admin processes — Run admin/management tasks as one-off processes

```
✅ Database migrations run as Kubernetes Jobs, not on app startup
✅ Data fixes run as one-off pods with the same codebase/image

# Flyway migration as a Kubernetes Job
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      containers:
      - name: migration
        image: myapp:v1.2.3
        command: ["java", "-jar", "app.jar", "--spring.flyway.enabled=true"]
      restartPolicy: Never
```

---

## 3. Beyond 12-Factor: The 15-Factor App

Three additional factors for modern cloud-native applications:

### Factor 13: API First
- Design the API before writing implementation code
- Use OpenAPI/Swagger specifications
- Enable parallel development of frontend and backend
- Contract-first approach

### Factor 14: Telemetry
- Application performance monitoring
- Domain-specific metrics (business KPIs)
- Health and system metrics
- Distributed tracing

### Factor 15: Authentication & Authorization
- Token-based authentication (JWT, OAuth 2.0)
- Never build your own auth — use identity providers
- Role-based and attribute-based access control
- Zero-trust: verify every request

---

## 4. Cloud-Native Maturity Model

Where does your application sit?

```
Level 0: CLOUD HOSTILE
├── Hardcoded IPs, file-system dependencies
├── Manual deployment
└── SAP Commerce on-premise (traditional)

Level 1: CLOUD READY
├── Externalized config
├── Uses managed services (DB, cache)
├── Containerized but still monolithic
└── SAP Commerce on Cloud Foundry

Level 2: CLOUD FRIENDLY
├── Follows 12-factor principles
├── Horizontal scaling works
├── CI/CD pipeline in place
├── Health checks and basic monitoring
└── SAP CAP applications

Level 3: CLOUD NATIVE
├── Microservices architecture
├── Event-driven communication
├── Full observability (metrics, traces, logs)
├── Infrastructure as Code
├── Auto-scaling based on metrics
└── Target architecture

Level 4: CLOUD NATIVE + SERVERLESS
├── Function-as-a-Service where appropriate
├── Event-driven at core
├── Pay-per-use economics
├── Minimal operational overhead
└── Advanced cloud-native
```

---

## 5. The CNCF Landscape

The [CNCF landscape](https://landscape.cncf.io/) is overwhelming. Here's a curated map of the most important categories:

```
┌───────────────────────────────────────────────────────┐
│                   CNCF LANDSCAPE                       │
├──────────────────┬────────────────────────────────────┤
│ ORCHESTRATION    │ Kubernetes, Docker Swarm, Nomad     │
├──────────────────┼────────────────────────────────────┤
│ RUNTIME          │ containerd, CRI-O, gVisor           │
├──────────────────┼────────────────────────────────────┤
│ NETWORKING       │ Envoy, Cilium, CoreDNS, Istio       │
├──────────────────┼────────────────────────────────────┤
│ STORAGE          │ etcd, Rook, Longhorn, MinIO          │
├──────────────────┼────────────────────────────────────┤
│ OBSERVABILITY    │ Prometheus, Grafana, Jaeger, Fluentd │
├──────────────────┼────────────────────────────────────┤
│ CI/CD            │ ArgoCD, Flux, Tekton, Jenkins X      │
├──────────────────┼────────────────────────────────────┤
│ SECURITY         │ OPA, Falco, Cert-Manager, SPIFFE     │
├──────────────────┼────────────────────────────────────┤
│ MESSAGING        │ NATS, CloudEvents, gRPC              │
└──────────────────┴────────────────────────────────────┘
```

**Key graduated projects** (production-ready, widely adopted):
- **Kubernetes** — container orchestration
- **Prometheus** — monitoring/alerting
- **Envoy** — service proxy
- **containerd** — container runtime
- **CoreDNS** — service discovery
- **etcd** — distributed key-value store
- **Helm** — package management
- **ArgoCD** — GitOps
- **Fluentd** — log collection
- **Jaeger** — distributed tracing
- **Open Policy Agent** — policy enforcement

---

## 6. Evolution: Monolith → SOA → Microservices → Serverless

### The Architecture Evolution

```
MONOLITH (1990s-2000s)
│  Everything in one deployable unit
│  One database, one team, one release cycle
│  Example: SAP Commerce / Hybris
│
├──→ SOA - Service Oriented Architecture (2000s-2010s)
│    │  Enterprise Service Bus (ESB)
│    │  SOAP/WSDL web services
│    │  Shared data models (canonical data model)
│    │  Still centralized governance
│    │  Example: SAP PI/PO, IBM WebSphere
│    │
│    ├──→ MICROSERVICES (2010s-present)
│    │    │  Small, independent services
│    │    │  Decentralized governance
│    │    │  Each service owns its data
│    │    │  Smart endpoints, dumb pipes
│    │    │  Example: Netflix, Spotify, most modern SaaS
│    │    │
│    │    ├──→ SERVERLESS / FaaS (2015s-present)
│    │    │    Event-triggered functions
│    │    │    No server management
│    │    │    Pay per execution
│    │    │    Example: AWS Lambda, Azure Functions
│    │    │
│    │    └──→ SERVICE MESH (2017s-present)
│    │         Infrastructure layer for service-to-service
│    │         Sidecar proxy pattern (Envoy)
│    │         Example: Istio, Linkerd
```

### SOA vs. Microservices — Key Differences

| Aspect | SOA | Microservices |
|--------|-----|---------------|
| Communication | ESB (smart pipes) | REST/gRPC/events (dumb pipes) |
| Data | Shared database common | Database per service |
| Governance | Centralized | Decentralized |
| Size | Large services | Small, focused services |
| Deployment | Often bundled | Independent deployment |
| Protocol | SOAP/XML heavy | REST/JSON, gRPC, events |
| Reuse | Via shared services | Via APIs |

---

## 7. Cloud-Native vs. Cloud-Hosted vs. Cloud-Ready

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  CLOUD-HOSTED (Lift & Shift)                                 │
│  ├── Same application, running on cloud VMs                  │
│  ├── Minimal changes to code                                 │
│  ├── Still a monolith                                        │
│  ├── Benefits: CapEx → OpEx, basic elasticity                │
│  └── Example: SAP Commerce on AWS EC2                        │
│                                                              │
│  CLOUD-READY (Lift, Shift & Optimize)                        │
│  ├── Containerized application                               │
│  ├── Uses some managed services (RDS, ElastiCache)           │
│  ├── Externalized configuration                              │
│  ├── Auto-scaling configured                                 │
│  ├── Benefits: better resource utilization, managed ops       │
│  └── Example: SAP Commerce on Kubernetes, CAP on CF          │
│                                                              │
│  CLOUD-NATIVE (Re-architect)                                 │
│  ├── Microservices architecture                              │
│  ├── Event-driven communication                              │
│  ├── Full observability                                      │
│  ├── CI/CD and GitOps                                        │
│  ├── Resilience patterns (circuit breakers, retries)         │
│  ├── Infrastructure as Code                                  │
│  ├── Benefits: max agility, resilience, scalability          │
│  └── Example: Purpose-built microservices on Kubernetes      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Migration Strategy Decision Matrix

| Factor | Lift & Shift | Optimize | Re-architect |
|--------|-------------|----------|--------------|
| **Timeline** | Weeks | Months | Quarters-Years |
| **Cost (short-term)** | Low | Medium | High |
| **Cost (long-term)** | High | Medium | Low |
| **Risk** | Low | Medium | High |
| **Agility gain** | Minimal | Moderate | Maximum |
| **When to choose** | Data center exit, quick migration | Good enough for now, incremental | Strategic investment, high change rate |

---

## 8. Exercises

### Exercise 1: 12-Factor Audit

Take a Java application you've worked on (e.g., from your SAP Commerce experience) and audit it against all 12 factors. For each factor:
1. Rate compliance: ✅ Compliant / 🟡 Partially / ❌ Non-compliant
2. Describe what would need to change to achieve compliance
3. Estimate the effort (Low / Medium / High)

**Deliverable:** A 12-factor compliance table with remediation plan.

### Exercise 2: Cloud-Native Maturity Assessment

Using the maturity model from Section 4, assess two systems:
1. A traditional SAP Commerce deployment
2. Your SAP CAP tutorial application

For each level (0-4), document:
- Which criteria are met
- Which are not met
- What changes would move the application to the next level

**Deliverable:** A maturity comparison document.

### Exercise 3: Architecture Decision — When NOT to Go Cloud-Native

Design a decision framework for when cloud-native is NOT the right approach. Consider:
- Small team (2-3 developers)
- Application with < 100 users
- Regulatory requirements for data sovereignty
- Application with no need for elastic scaling
- Tight budget and timeline

**Deliverable:** A decision tree or flowchart with justification.

### Exercise 4: Hands-On — 12-Factor Spring Boot App

Create a Spring Boot application that demonstrably follows all 12 factors:
1. Create a simple REST API (e.g., a task manager)
2. Externalize all configuration via environment variables
3. Add structured JSON logging to stdout
4. Add health check endpoints (`/actuator/health`)
5. Create a `Dockerfile` with multi-stage build
6. Create a `docker-compose.yml` with PostgreSQL and Redis
7. Implement graceful shutdown

**Deliverable:** A working GitHub repository.

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

10. What level of the cloud-native maturity model would you assign to: (a) a VM-based Hybris deployment, (b) a containerized Spring Boot app with CI/CD, (c) a serverless event-driven architecture?

---

## 10. Answers

### Answer 1
The four pillars are:
1. **Microservices** — small, independent, focused services
2. **Containers** — lightweight, portable runtime environments
3. **CI/CD** — automated build, test, and deployment pipelines
4. **DevOps** — culture of shared ownership between dev and ops

### Answer 2
- **Cloud-hosted:** A traditional monolithic application running on cloud VMs with minimal code changes. Example: SAP Commerce deployed on AWS EC2 instances — same WAR file, same architecture, just different hardware.
- **Cloud-native:** An application built from the ground up to leverage cloud capabilities — microservices, containers, event-driven, auto-scaling. Example: A microservices-based e-commerce platform on Kubernetes with each service independently deployable and scalable.

The key difference: cloud-hosted is about *where* it runs, cloud-native is about *how* it's built.

### Answer 3
This violates **Factor 3: Config** (store config in the environment).

Problems:
- Production URL and API key are hardcoded
- API key (a secret) is in source code — security risk
- Changing the URL requires code change and redeployment

Fix:
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
In Kubernetes, the API key should be stored in a Secret, not a ConfigMap.

### Answer 4
Reasons to write logs to stdout:
1. **Separation of concerns** — the application generates logs, the platform (Kubernetes, Cloud Foundry) is responsible for collection, aggregation, and routing
2. **Portability** — the same application works in Docker, Kubernetes, Cloud Foundry, or bare metal without changing log configuration
3. **No disk management** — no need to worry about log rotation, disk space, or file permissions
4. **Stream processing** — stdout can be captured and forwarded in real-time to centralized logging (ELK, Fluentd, CloudWatch)
5. **Container-friendly** — `docker logs` and `kubectl logs` read from stdout/stderr

### Answer 5
**Problem:** With sticky sessions disabled (default), requests from the same user may hit different instances. Each instance has its own in-memory session store, so the user's session data (cart, login state) will be lost or inconsistent.

**Solution 1:** External session store — use Spring Session with Redis:
```java
@EnableRedisHttpSession
public class SessionConfig { }
```
All instances share the same session store in Redis.

**Solution 2:** Stateless design — store no session on the server. Use JWT tokens that contain all necessary state. The client sends the token with every request, so any instance can handle any request.

**Why not sticky sessions?** While possible, sticky sessions (load balancer affinity) defeat the purpose of horizontal scaling — if the "sticky" instance goes down, the session is lost. It also creates uneven load distribution.

### Answer 6

| Aspect | SOA | Microservices |
|--------|-----|---------------|
| **Communication** | ESB (Enterprise Service Bus) — smart routing, transformation, orchestration in the middleware | Direct service-to-service via REST/gRPC or async messaging — "smart endpoints, dumb pipes" |
| **Data** | Typically shared database or canonical data model | Database per service — each service owns its data |
| **Governance** | Centralized — shared schemas, enterprise standards | Decentralized — each team chooses its own tech stack and data model |

Additional differences:
- SOA services tend to be large and cover broad business domains; microservices are small and focused
- SOA emphasizes reuse through shared services; microservices prefer duplication over coupling
- SOA uses SOAP/XML heavily; microservices prefer REST/JSON or gRPC

### Answer 7
Questions to ask before recommending microservices:
1. **Why?** What problem are you trying to solve? Slow deployments? Scaling issues? Team autonomy?
2. **Team size?** Do you have enough teams (3+ teams of 5-8 people) to own separate services? Microservices without team ownership is a distributed monolith.
3. **Domain maturity?** Do you understand the domain boundaries well? Premature decomposition is worse than a monolith.
4. **Operational readiness?** Do you have CI/CD, monitoring, distributed tracing, container orchestration? Microservices without operational maturity is chaos.
5. **Can you start incrementally?** Instead of a big rewrite, can you use the Strangler Fig pattern to extract services gradually?
6. **What's the deployment frequency goal?** If you deploy once a month, microservices may not help much.
7. **Is the team comfortable with distributed systems complexity?** Distributed transactions, eventual consistency, network failures.

### Answer 8
The three additional factors (15-factor app):
1. **API First** — design APIs before implementation, use contract-first approach
2. **Telemetry** — built-in monitoring, metrics, health checks, distributed tracing
3. **Authentication & Authorization** — token-based auth (OAuth 2.0/JWT), never build your own, zero-trust

### Answer 9
**Cattle, not pets:**
- **Pets:** Servers have names (prod-server-01), are cared for individually, get patched and nursed back to health when sick. If one dies, everyone panics.
- **Cattle:** Servers have numbers (instance-4a7f), are identical and interchangeable. If one is sick, you kill it and replace it with a new one. Nobody cares about individual instances.

**Relationship to immutable infrastructure:**
Immutable infrastructure means you never modify a running server. Instead:
1. Build a new image with the fix/update
2. Deploy new instances with the new image
3. Kill old instances

This only works if servers are "cattle" — you must be able to destroy and recreate any instance at any time. If you have "pets" with unique configurations, you can't simply replace them.

**Example:** Instead of SSH-ing into a production server to apply a security patch (pets), you:
1. Update the Dockerfile
2. Build a new container image
3. Rolling-update the Kubernetes deployment
4. Kubernetes replaces old pods with new ones

### Answer 10
- **(a) VM-based Hybris deployment:** **Level 0-1 (Cloud Hostile to Cloud Ready)**
  - Level 0 if on-premises with hardcoded configs
  - Level 1 if migrated to cloud VMs with externalized DB config
  - Still a monolith, likely manual deployments, limited scaling

- **(b) Containerized Spring Boot app with CI/CD:** **Level 2 (Cloud Friendly)**
  - Containerized ✅
  - CI/CD pipeline ✅
  - Likely follows most 12-factor principles
  - May still be a single service (not microservices)
  - Missing: full observability, event-driven communication, auto-scaling

- **(c) Serverless event-driven architecture:** **Level 4 (Cloud Native + Serverless)**
  - Event-driven at core ✅
  - Functions as a Service ✅
  - Pay per execution ✅
  - Minimal operational overhead ✅
  - Highest maturity level

---

## Key Takeaways

1. **Cloud-native is a mindset**, not a technology. It's about designing for the cloud's strengths (elasticity, managed services, automation).
2. **The 12-Factor methodology** is your checklist — audit every application against it.
3. **Not everything needs to be cloud-native.** The maturity model helps you choose the right level for each workload.
4. **Your SAP experience is valuable.** Understanding monolith pain points makes you better at designing cloud-native solutions. The anti-patterns you've seen become the motivation for cloud-native principles.
5. **Start with the strangler fig pattern** — don't do big-bang rewrites.

---

*Next lesson: [02 - Architecture Styles & Patterns](02-architecture-styles.md)*