# Prompt: Lesson 07 — API Design & Management

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 07: API Design & Management**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot. Reference: Microsoft API Design Best Practices (https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design).

### Topic Coverage

1. **REST API Design**
   - Richardson Maturity Model (4 levels, why target Level 2)
   - Design best practices: resources (nouns not verbs), nested resources, filtering/sorting/pagination
   - Error responses following RFC 7807 (Problem Details) with JSON example
   - HTTP status codes table (12 essential codes with when-to-use)
   - Pagination: offset-based vs cursor-based with JSON response examples
   - Idempotency: safe methods, Idempotency-Key header for POST

2. **gRPC**
   - When to use gRPC vs REST (comparison table)
   - Protocol Buffers definition example (OrderService with CreateOrder, GetOrder, WatchOrderStatus)
   - Communication patterns table (Unary, Server streaming, Client streaming, Bidirectional)

3. **GraphQL**
   - Query example showing no over/under-fetching
   - When to use vs avoid (table)
   - GraphQL Federation for microservices

4. **API Versioning**
   - Three strategies comparison table (URL path, header, query param)
   - Backward compatibility rules (safe vs breaking changes)

5. **API Security**
   - OAuth 2.0 + OpenID Connect flow diagram
   - JWT structure and validation steps
   - Stateless authentication in microservices

6. **API Gateway Patterns**
   - Gateway routing
   - Gateway offloading (6 cross-cutting concerns)
   - Gateway aggregation

7. **Consumer-Driven Contract Testing**
   - Pact workflow (consumer writes contract → provider verifies)
   - Java Pact test code example
   - OpenAPI/Swagger YAML example

### Code Examples Required
1. RFC 7807 error response JSON
2. REST resource design examples (correct vs incorrect)
3. gRPC Protocol Buffer definition
4. GraphQL query example
5. Pact consumer test (Java)
6. OpenAPI specification YAML
7. Idempotency-Key implementation pattern

### Diagrams Required (minimum 5)
Richardson Maturity Model levels, OAuth 2.0 flow, JWT validation in microservices, API Gateway with routing, Contract testing workflow

### Real-World Case Study
Design API strategy for an e-commerce platform: which APIs are REST, which are gRPC, how versioning works, gateway configuration, contract testing setup.

### Exercises (3)
1. Design REST API for library management (15 endpoints, RFC 7807, pagination, OpenAPI)
2. Versioning strategy for breaking change (customer field migration)
3. gRPC vs REST decision for 5 scenarios

### Self-Check Questions (7)
Cover: Richardson Maturity Model, HTTP status codes, idempotency, gRPC vs REST, contract testing, versioning strategies, JWT authentication.

### References
Include: Microsoft API Design Best Practices, "RESTful Web APIs" (Richardson & Amundsen), OpenAPI specification, Pact documentation, gRPC documentation.