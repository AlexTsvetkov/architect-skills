# Lesson 07: API Design & Management

> **Prerequisites:** Lessons 01-06  
> **Time estimate:** 6-8 hours  
> **Parallel reading:** "Building Microservices" — Sam Newman (Ch 7-10)

---

## Table of Contents

1. [REST API Design](#1-rest-api-design)
2. [gRPC](#2-grpc)
3. [GraphQL](#3-graphql)
4. [API Versioning](#4-api-versioning)
5. [API Security](#5-api-security)
6. [API Gateway Patterns](#6-api-gateway-patterns)
7. [Consumer-Driven Contract Testing](#7-contract-testing)
8. [Exercises](#8-exercises)
9. [Self-Check Questions](#9-self-check-questions)
10. [Answers](#10-answers)

---

## 1. REST API Design

### Richardson Maturity Model

```
Level 3: Hypermedia Controls (HATEOAS)
  → Responses include links to related actions/resources
  → Client discovers capabilities from API responses
  
Level 2: HTTP Verbs + Status Codes  ← TARGET THIS
  → Proper use of GET/POST/PUT/PATCH/DELETE
  → Proper HTTP status codes (200, 201, 400, 404, 409, 500)
  
Level 1: Resources
  → Individual URIs for each resource (/orders/123)
  
Level 0: Single endpoint (RPC-style)
  → POST /api with action in body
```

**Target Level 2** for most APIs. Level 3 (HATEOAS) adds complexity with limited practical benefit in most microservices architectures.

### Design Best Practices

```
Resources (nouns, not verbs):
  ✅ GET /orders         — list orders
  ✅ GET /orders/123     — get order 123
  ✅ POST /orders        — create order
  ✅ PUT /orders/123     — replace order 123
  ✅ PATCH /orders/123   — partial update order 123
  ✅ DELETE /orders/123  — delete order 123
  
  ❌ GET /getOrders
  ❌ POST /createOrder
  ❌ POST /deleteOrder/123

Nested resources:
  GET /orders/123/items         — items in order 123
  POST /orders/123/items        — add item to order 123
  GET /customers/456/orders     — orders for customer 456

Filtering, sorting, pagination:
  GET /orders?status=shipped&sort=-createdAt&page=2&size=20
```

### Error Responses (RFC 7807 — Problem Details)

```json
{
  "type": "https://api.myshop.com/errors/insufficient-stock",
  "title": "Insufficient Stock",
  "status": 409,
  "detail": "Product PROD-123 has only 2 units available, 5 requested",
  "instance": "/orders/789",
  "productId": "PROD-123",
  "available": 2,
  "requested": 5
}
```

### HTTP Status Codes to Know

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST (include Location header) |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input, validation errors |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | State conflict (concurrent modification, business rule violation) |
| 422 | Unprocessable Entity | Semantically invalid (valid JSON but business logic fails) |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server failure |
| 503 | Service Unavailable | Temporary overload or maintenance |

### Pagination

```json
// Offset-based (simple but slow for large offsets)
GET /orders?page=3&size=20

{
  "content": [...],
  "page": 3,
  "size": 20,
  "totalElements": 1250,
  "totalPages": 63
}

// Cursor-based (efficient for large datasets)
GET /orders?after=eyJpZCI6MTIzfQ&limit=20

{
  "data": [...],
  "cursors": {
    "after": "eyJpZCI6MTQzfQ",
    "hasMore": true
  }
}
```

### Idempotency

```
Idempotent methods (safe to retry):
  GET, PUT, DELETE, HEAD — same result if called once or many times

Non-idempotent:
  POST — creates a new resource each time

Making POST idempotent (Idempotency Key):
  POST /payments
  Idempotency-Key: abc-123-def

  Server stores: key → result
  If same key sent again → return cached result (don't charge twice!)
```

---

## 2. gRPC

### When to Use gRPC

| Use gRPC For | Stick with REST For |
|-------------|-------------------|
| Internal service-to-service | Public-facing APIs |
| High performance / low latency | Browser clients |
| Streaming data | Simple CRUD operations |
| Polyglot environments (code gen) | Broad client compatibility |
| Strict API contracts | When human readability matters |

### Protocol Buffers

```protobuf
// order.proto
syntax = "proto3";
package ecommerce;

service OrderService {
  rpc CreateOrder (CreateOrderRequest) returns (OrderResponse);
  rpc GetOrder (GetOrderRequest) returns (OrderResponse);
  rpc WatchOrderStatus (GetOrderRequest) returns (stream OrderStatusUpdate);
}

message CreateOrderRequest {
  string customer_id = 1;
  repeated OrderItem items = 2;
}

message OrderItem {
  string product_id = 1;
  int32 quantity = 2;
}

message OrderResponse {
  string order_id = 1;
  string status = 2;
  double total = 3;
}
```

### gRPC Communication Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| Unary | Single request → single response | CreateOrder |
| Server streaming | Single request → stream of responses | WatchOrderStatus |
| Client streaming | Stream of requests → single response | UploadItems |
| Bidirectional | Stream both ways | Chat, real-time sync |

---

## 3. GraphQL

### Overview

Query language for APIs. Client specifies exactly what data it needs.

```graphql
# Client requests only what it needs
query {
  order(id: "123") {
    id
    status
    items {
      productName
      quantity
    }
    customer {
      name
      email
    }
  }
}

# No over-fetching (getting fields you don't need)
# No under-fetching (needing multiple requests)
```

### When to Use vs. When to Avoid

| ✅ Good Fit | ❌ Poor Fit |
|------------|-----------|
| Multiple frontends with different data needs | Simple CRUD APIs |
| Complex, nested data relationships | Real-time streaming |
| Reducing over/under-fetching | Caching-critical (REST caches better) |
| Rapid frontend iteration | Small team with single frontend |

### GraphQL Federation

Multiple microservices expose GraphQL schemas, composed into one unified graph:

```
Client → Federated Gateway → Order Subgraph (Order Service)
                           → Product Subgraph (Product Service)
                           → Customer Subgraph (Customer Service)
```

---

## 4. API Versioning

### Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| URL path | `/api/v1/orders` | Simple, explicit | URL changes |
| Header | `Accept: application/vnd.myapi.v2+json` | Clean URLs | Less visible |
| Query param | `/api/orders?version=2` | Easy to test | Messy URLs |

**Recommendation:** URL path versioning for most cases. It's simple and explicit.

### Backward Compatibility Rules

```
SAFE changes (no version bump needed):
  ✅ Adding new optional fields to response
  ✅ Adding new endpoints
  ✅ Adding new optional query parameters

BREAKING changes (need new version):
  ❌ Removing fields from response
  ❌ Renaming fields
  ❌ Changing field types
  ❌ Making optional fields required
  ❌ Changing URL structure
```

---

## 5. API Security

### OAuth 2.0 + OpenID Connect

```
┌────────┐  1. Login  ┌───────────────┐
│  User  │──────────→ │ Identity      │
│        │←──────────│ Provider      │
│        │ 2. Tokens  │ (Keycloak,    │
└───┬────┘           │  Auth0)       │
    │                 └───────────────┘
    │ 3. API call with
    │    Bearer token
    ▼
┌──────────┐  4. Validate  ┌──────────────┐
│  API     │  token ──────→│ Token        │
│  Gateway │←──────────────│ Validation   │
└──────────┘               └──────────────┘
```

### JWT Structure

```
Header.Payload.Signature

Payload contains:
{
  "sub": "user-123",           // Subject
  "roles": ["admin", "user"],   // Roles
  "exp": 1711234567,            // Expiration
  "iss": "https://auth.myshop.com"  // Issuer
}

Services validate JWT locally (no call to auth server)
→ Stateless authentication
→ Each service can check roles/permissions
```

---

## 6. API Gateway Patterns

### Gateway Routing

```
api.myshop.com/orders/*    → order-service
api.myshop.com/products/*  → product-service
api.myshop.com/payments/*  → payment-service
```

### Gateway Offloading

Move cross-cutting concerns from services to the gateway:
- SSL termination
- Authentication / JWT validation
- Rate limiting
- Request/response logging
- CORS handling
- Compression

### Gateway Aggregation

Combine multiple backend calls into a single client request (covered in Lesson 06).

---

## 7. Contract Testing

### Consumer-Driven Contracts with Pact

```
1. Consumer (Order Service) writes a Pact test:
   "When I call GET /inventory/PROD-123
    I expect: { productId: string, available: number, reserved: number }"

2. Pact generates a contract file (JSON)

3. Provider (Inventory Service) runs verification:
   "Does my API satisfy the contract from Order Service?"

4. If verification fails → provider build fails
   → provider knows they would break Order Service
```

```java
// Consumer test (Order Service)
@Pact(consumer = "OrderService")
public V4Pact inventoryPact(PactDslWithProvider builder) {
    return builder
        .given("product PROD-123 exists")
        .uponReceiving("get inventory for PROD-123")
        .path("/inventory/PROD-123")
        .method("GET")
        .willRespondWith()
        .status(200)
        .body(newJsonBody(body -> {
            body.stringType("productId", "PROD-123");
            body.integerType("available", 50);
        }).build())
        .toPact(V4Pact.class);
}
```

### OpenAPI / Swagger

```yaml
openapi: 3.0.3
info:
  title: Order Service API
  version: 1.0.0
paths:
  /orders:
    post:
      summary: Create a new order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        '201':
          description: Order created
          headers:
            Location:
              schema: { type: string }
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderResponse'
```

---

## 8. Exercises

### Exercise 1: API Design

Design a REST API for a library management system:
1. Resources: books, members, loans, reservations
2. At least 15 endpoints
3. Error responses following RFC 7807
4. Pagination strategy
5. Create an OpenAPI specification

### Exercise 2: Versioning Strategy

Your Order Service API currently returns:
```json
{ "id": "123", "total": 99.99, "customer": "Alice", "status": "shipped" }
```
You need to change `customer` from a string to an object `{ "name": "Alice", "email": "alice@..." }`.

Design a backward-compatible migration strategy with: versioning approach, migration timeline, client communication plan.

### Exercise 3: gRPC vs REST Decision

For each scenario, choose REST or gRPC and justify:
1. Public API consumed by mobile apps
2. Real-time price feed between trading services
3. Catalog search API consumed by web frontend
4. Internal log aggregation between services
5. Webhook notifications to external partners

---

## 9. Self-Check Questions

1. What is the Richardson Maturity Model? Which level should you target and why?
2. Name five HTTP status codes and when to use each.
3. What is idempotency and why is it important for APIs?
4. When would you choose gRPC over REST?
5. What is consumer-driven contract testing? Why is it essential for microservices?
6. Explain three API versioning strategies with pros and cons.
7. How does JWT-based authentication work in a microservices architecture?

---

## 10. Answers

### Answer 1
The Richardson Maturity Model defines four levels: Level 0 (single endpoint RPC), Level 1 (individual resources with URIs), Level 2 (proper HTTP verbs and status codes), Level 3 (HATEOAS with hypermedia controls). Target **Level 2**: it provides a well-structured, standards-compliant API. Level 3 (HATEOAS) is theoretically elegant but adds significant complexity. In microservices, clients typically know the API structure through documentation/contracts, making HATEOAS less valuable.

### Answer 2
- **200 OK:** Successful GET, PUT, PATCH requests
- **201 Created:** Successful POST that created a resource (include Location header)
- **400 Bad Request:** Client sent invalid data (malformed JSON, missing required fields)
- **404 Not Found:** Requested resource doesn't exist
- **409 Conflict:** Business rule violation or concurrent modification conflict

### Answer 3
An idempotent operation produces the same result whether called once or multiple times. Important because in distributed systems, network failures may cause retries. If POST /payments is retried, you don't want to charge twice. Solution: use an Idempotency-Key header. Server stores key→result mapping; duplicate requests return the cached result.

### Answer 4
Choose gRPC when: (1) internal service-to-service where browser compatibility doesn't matter, (2) high performance / low latency requirements (10x faster than REST), (3) streaming data (bidirectional streaming native), (4) polyglot environments benefiting from Protobuf code generation, (5) strict API contracts needed (Protobuf schema enforcement).

### Answer 5
Consumer-driven contract testing: the consumer of an API defines what it expects (contract). The provider verifies its API satisfies all consumer contracts. Essential because: catches breaking changes before production, enables independent deployment (each team deploys confidently), no need for shared integration environments, contracts serve as living documentation. Tool: Pact.

### Answer 6
**URL path** (`/api/v1/orders`): Pros — simple, explicit, easy to route. Cons — URL changes, harder to maintain multiple versions. **Header** (`Accept: vnd.api.v2+json`): Pros — clean URLs, content negotiation. Cons — less visible, harder to test. **Query param** (`?version=2`): Pros — easy to add. Cons — messy, not standard.

### Answer 7
User authenticates with Identity Provider (IdP), receives a JWT. Client includes JWT as `Authorization: Bearer <token>` header in API calls. API Gateway (or each service) validates the JWT by checking: signature (using IdP's public key), expiration, issuer, audience. No call to IdP needed for validation (stateless). The JWT payload contains user identity, roles, permissions. Each service extracts roles from JWT to make authorization decisions.

---

*Previous: [06 - Cloud-Native Design Patterns](06-cloud-native-patterns.md)*  
*Next: [08 - Observability & Reliability](08-observability-reliability.md)*