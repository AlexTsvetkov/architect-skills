# Prompt: Lesson 01 — Cloud-Native Architecture Fundamentals

```markdown
You are an expert cloud-native software architect and technical educator. Your task is to create **Lesson 01: Cloud-Native Architecture Fundamentals** for a professional-level course on cloud-native architecture.

### Context about the course

This course targets experienced Java developers (5+ years) transitioning to architect roles. They have strong Spring Boot experience, enterprise monolith background, and have read "System Design Interview" by Alex Xu. Lessons must be **deep, technical, and practical** — not surface-level overviews. Primary stack: **Java / Spring Boot**.

### Topic: Cloud-Native Architecture Fundamentals

Cover the following areas in depth:

1. **What "Cloud-Native" Really Means**
   - CNCF official definition vs. practical reality
   - The four pillars: microservices, containers, CI/CD, DevOps
   - Key principles: design for failure, cattle not pets, immutable infrastructure, declarative configuration, loose coupling
   - Connecting enterprise monolith experience to cloud-native concepts

2. **The 12-Factor App Methodology**
   - Deep dive into ALL 12 factors with Java/Spring Boot code examples for each
   - For each factor: what it is, why it matters, Spring Boot implementation, anti-pattern example
   - Special focus on: config externalization, stateless processes, backing services, logging to stdout

3. **Beyond 12-Factor: The 15-Factor App**
   - Factor 13: API First
   - Factor 14: Telemetry
   - Factor 15: Authentication & Authorization

4. **Cloud-Native Maturity Model**
   - 5 levels: Cloud Hostile → Cloud Ready → Cloud Friendly → Cloud Native → Cloud Native + Serverless
   - Characteristics of each level
   - How to assess your application's current level

5. **The CNCF Landscape**
   - Curated overview of key categories (not exhaustive)
   - Graduated projects and why they matter
   - How to navigate the landscape

6. **Evolution: Monolith → SOA → Microservices → Serverless**
   - Historical context and driving forces
   - SOA vs. Microservices comparison table (communication, data, governance, size, deployment, protocol)
   - Service mesh evolution

7. **Cloud-Native vs. Cloud-Hosted vs. Cloud-Ready**
   - Clear definitions with examples
   - Migration strategy decision matrix (timeline, cost, risk, agility)

### Code Examples Required (Java/Spring Boot)

1. **12-Factor Config violation and fix** — Hardcoded credentials vs. externalized config with `@Value` and environment variables
2. **Stateful vs. Stateless process** — In-memory session anti-pattern vs. Redis-backed sessions with `@EnableRedisHttpSession`
3. **Graceful shutdown** — `@PreDestroy` lifecycle hook + `application.yml` configuration for graceful shutdown
4. **Structured logging to stdout** — logback-spring.xml JSON configuration
5. **Docker Compose for dev/prod parity** — Complete docker-compose.yml with app, PostgreSQL, Redis, Kafka

### Diagrams Required

1. Four pillars of cloud-native (ASCII table)
2. Build → Release → Run pipeline flow
3. Scale-out process model (load balancer + pods)
4. Architecture evolution tree (Monolith → SOA → Microservices → Serverless)
5. Cloud-Native vs Cloud-Hosted vs Cloud-Ready comparison
6. CNCF Landscape curated map

### Exercises (4)

1. **12-Factor Audit** — Audit a Java application against all 12 factors with compliance rating
2. **Maturity Assessment** — Compare two systems using the maturity model
3. **When NOT to Go Cloud-Native** — Design a decision framework
4. **Hands-On: 12-Factor Spring Boot App** — Build a complete app following all 12 factors

### Self-Check Questions (10)

Include questions about:
- Four pillars identification
- Cloud-native vs cloud-hosted distinction
- 12-Factor violation identification (code-based)
- Logging to stdout rationale
- Session state scaling problem
- SOA vs microservices differences
- Migration decision advice
- 15-factor additional factors
- Cattle vs pets principle
- Maturity model level assignment

### References

Include: 12factor.net, CNCF landscape, Microsoft Azure Architecture Center, Martin Fowler's blog, "Fundamentals of Software Architecture" by Richards & Ford.