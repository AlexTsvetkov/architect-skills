# Prompt: Lesson 06 — Cloud-Native Design Patterns

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 06: Cloud-Native Design Patterns**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot with Resilience4j. Reference: Microsoft Cloud Design Patterns (https://learn.microsoft.com/en-us/azure/architecture/patterns/).

### Topic Coverage

1. **Resilience Patterns**
   - **Circuit Breaker**: State diagram (CLOSED→OPEN→HALF-OPEN), Resilience4j code + YAML config, fallback method
   - **Retry with Exponential Backoff and Jitter**: Timing diagram, why jitter matters (thundering herd), Resilience4j code
   - **Bulkhead**: Thread pool isolation diagram (with/without), Resilience4j code
   - **Timeout**: TimeLimiter with CompletableFuture, YAML config
   - **Health Endpoint Monitoring**: Custom HealthIndicator, K8s liveness/readiness integration
   - **Rate Limiting / Throttling**: RateLimiter with fallback returning 429

2. **Structural Patterns**
   - **Sidecar Pattern**: Pod diagram (main + sidecar sharing network/storage), 4 examples (Envoy, Fluentd, Vault Agent, config reloader)
   - **Ambassador Pattern**: Outbound proxy for retries, circuit breaking, TLS
   - **Adapter Pattern**: Legacy metrics → Prometheus format conversion

3. **Migration Patterns**
   - **Strangler Fig Pattern**: 3-phase diagram, implementation approaches (API Gateway routing, CDC)
   - **Branch by Abstraction**: 6-step process with interface + feature flag code example
   - **Anti-Corruption Layer**: Diagram showing model translation between new and legacy

4. **Communication Patterns**
   - **Gateway Aggregation**: Fan-out to multiple services, compose response
   - **Competing Consumers**: Queue with multiple workers diagram
   - **Transactional Outbox**: Dual-write problem + solution (DB transaction + outbox table + relay)
   - **Valet Key Pattern**: Pre-signed URL flow (4 steps)

### Code Examples Required (all with Resilience4j)
1. Circuit breaker with fallback method + YAML configuration
2. Retry with exponential backoff + YAML configuration
3. Bulkhead (thread pool isolation) + configuration
4. TimeLimiter with CompletableFuture
5. Custom HealthIndicator for Spring Boot Actuator
6. RateLimiter with 429 fallback
7. Branch by Abstraction with feature flag (interface + two implementations + @Bean factory)
8. Transactional Outbox (SQL transaction with outbox insert)

### Diagrams Required (minimum 7)
Circuit breaker state machine, Bulkhead isolation (with/without), Sidecar pattern in K8s pod, Ambassador pattern, Strangler fig phases, Anti-corruption layer, Transactional outbox flow, Competing consumers

### Real-World Case Study
Design resilience architecture for a food delivery platform showing where each pattern applies.

### Exercises (3)
1. Resilience design for payment service (patterns, config values, fallbacks, chaos testing)
2. Migration plan for 500K LOC monolith (pattern selection, extraction order, shared DB, data sync)
3. Pattern combination for food delivery platform (use 5+ patterns with justification)

### Self-Check Questions (7)
Cover: circuit breaker states, jitter importance, bulkhead purpose, sidecar examples, strangler fig process, transactional outbox, BFF pattern.

### References
Include: Microsoft Cloud Design Patterns, "Cloud Native Patterns" (Cornelia Davis), "Release It!" (Michael Nygard), Resilience4j docs.