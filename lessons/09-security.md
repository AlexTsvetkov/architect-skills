# Lesson 09: Security in Cloud-Native Systems

> **Prerequisites:** Lessons 01-08  
> **Time estimate:** 6-8 hours

---

## Table of Contents

1. [Zero-Trust Architecture](#1-zero-trust-architecture)
2. [Identity and Access Management](#2-identity-and-access-management)
3. [OAuth 2.0 / OpenID Connect Deep Dive](#3-oauth-20--openid-connect)
4. [Service-to-Service Authentication](#4-service-to-service-authentication)
5. [Secret Management](#5-secret-management)
6. [Container Security](#6-container-security)
7. [Network Security](#7-network-security)
8. [Supply Chain Security](#8-supply-chain-security)
9. [Exercises](#9-exercises)
10. [Self-Check Questions](#10-self-check-questions)
11. [Answers](#11-answers)

---

## 1. Zero-Trust Architecture

### Principle: "Never Trust, Always Verify"

```
Traditional (perimeter security):
  Outside ──[firewall]──→ Inside (trusted)
  Once inside, everything trusts everything.

Zero-Trust:
  Every request is verified, regardless of source.
  Every service authenticates and authorizes every call.
  No implicit trust based on network location.
```

### Zero-Trust Pillars

| Pillar | Implementation |
|--------|---------------|
| Verify identity | mTLS between services, JWT for users |
| Least privilege | RBAC, service accounts with minimal permissions |
| Assume breach | Encrypt everything, segment network, monitor all access |
| Verify explicitly | Authenticate and authorize every request |
| Micro-segmentation | Network policies, service mesh access control |

---

## 2. Identity and Access Management

### Authentication vs. Authorization

```
Authentication (AuthN): WHO are you?
  → Username/password, tokens, certificates
  → "This is user alice@example.com"

Authorization (AuthZ): WHAT can you do?
  → Roles, permissions, policies
  → "alice can read orders but not delete them"
```

### RBAC vs. ABAC

| Model | Description | Example |
|-------|-------------|---------|
| **RBAC** (Role-Based) | Permissions assigned to roles | admin role → full access |
| **ABAC** (Attribute-Based) | Policies based on attributes | If department=finance AND time=business_hours → access |
| **ReBAC** (Relationship-Based) | Based on resource relationships | User can edit documents they created |

---

## 3. OAuth 2.0 / OpenID Connect

### OAuth 2.0 Flows

#### Authorization Code Flow (Web Apps)

```
1. User → App: "I want to log in"
2. App → Auth Server: redirect to login page
3. User → Auth Server: enters credentials
4. Auth Server → App: authorization code (via redirect)
5. App → Auth Server: exchange code for tokens (server-side)
6. Auth Server → App: access_token + refresh_token + id_token

Most secure flow. Code exchange happens server-side.
```

#### Client Credentials Flow (Service-to-Service)

```
1. Service A → Auth Server: client_id + client_secret
2. Auth Server → Service A: access_token
3. Service A → Service B: request + Bearer access_token

No user involved. Machine-to-machine communication.
```

### JWT Token Validation

```
Each service validates JWT independently:
1. Check signature (using IdP's public key / JWKS endpoint)
2. Check expiration (exp claim)
3. Check issuer (iss claim)
4. Check audience (aud claim)
5. Extract roles/permissions from claims
6. Make authorization decision

No call to auth server needed → stateless, fast
```

### Token Architecture

```
┌─────────────┐
│ API Gateway  │ ← Validates access token
│              │ ← Rate limiting, SSL termination
└──────┬───────┘
       │ Forwards JWT (or internal token)
       ▼
┌──────────────┐
│ Order Service │ ← Extracts user roles from JWT
│              │ ← Makes authz decision
└──────┬───────┘
       │ Propagates JWT (or service token)
       ▼
┌──────────────┐
│ Inventory Svc│ ← Validates calling service identity
└──────────────┘
```

---

## 4. Service-to-Service Authentication

### mTLS (Mutual TLS)

```
Traditional TLS: Client verifies server
mTLS: Client AND server verify each other

Service A ←→ Service B
  Both present certificates
  Both validate each other's certificates
  Traffic is encrypted

Service mesh (Istio/Linkerd) handles mTLS automatically:
  Sidecar proxies manage certificates
  Certificate rotation is automatic
  Applications don't need to change
```

### JWT Propagation

```
User request → API Gateway (validates user JWT)
  → Order Service (extracts user context from JWT)
    → Payment Service (receives propagated JWT, validates, checks permissions)

Each service knows WHO the original user is and WHAT they can do.
```

---

## 5. Secret Management

### Don't Do This

```
❌ Secrets in source code
❌ Secrets in Docker images
❌ Secrets in environment variables (visible in process list)
❌ Secrets in ConfigMaps (not encrypted)
```

### Kubernetes Secrets (Minimum)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # base64 (NOT encryption!)

# Mount as volume or env var
# Enable encryption at rest in etcd
```

### HashiCorp Vault (Recommended)

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│ Application  │────→│ Vault Agent  │────→│ HashiCorp    │
│              │     │ (Sidecar)    │     │ Vault        │
│ Reads secrets│     │ Fetches and  │     │              │
│ from files   │     │ renews       │     │ Stores,      │
│              │     │ secrets      │     │ rotates,     │
└─────────────┘     └──────────────┘     │ audits       │
                                         └──────────────┘

Benefits:
- Dynamic secrets (generated on demand, auto-expire)
- Automatic rotation
- Audit trail (who accessed what, when)
- Fine-grained access policies
```

### Cloud-Native Secret Management

| Cloud | Service |
|-------|---------|
| AWS | Secrets Manager, Parameter Store |
| Azure | Key Vault |
| GCP | Secret Manager |
| Kubernetes | External Secrets Operator (syncs from cloud to K8s) |

---

## 6. Container Security

### Image Security

```
1. Use minimal base images (distroless, Alpine)
2. Scan for vulnerabilities: trivy image myapp:latest
3. Sign images (cosign / Docker Content Trust)
4. Use private registries with access control
5. Pin image digests (not just tags): image: myapp@sha256:abc123...
```

### Runtime Security

```yaml
# Pod Security Context
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: [ALL]
```

### Pod Security Standards

| Level | Description |
|-------|-------------|
| Privileged | No restrictions (avoid in production) |
| Baseline | Prevents known privilege escalations |
| Restricted | Heavily restricted (best for production) |

---

## 7. Network Security

### Network Policies

```yaml
# Default deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes: [Ingress]

---
# Allow only from specific services
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-order-to-payment
spec:
  podSelector:
    matchLabels: { app: payment-service }
  ingress:
  - from:
    - podSelector:
        matchLabels: { app: order-service }
    ports:
    - port: 8080
```

### Defense in Depth

```
Layer 1: Cloud firewall / security groups
Layer 2: Kubernetes Network Policies
Layer 3: Service mesh mTLS + access policies
Layer 4: Application-level authentication (JWT)
Layer 5: Application-level authorization (RBAC)
```

---

## 8. Supply Chain Security

### SBOM (Software Bill of Materials)

Document all components in your software: dependencies, base images, tools. Enables vulnerability tracking across the supply chain.

### Image Signing

```bash
# Sign with cosign
cosign sign --key cosign.key myregistry/myapp:v1.0.0

# Verify before deployment (admission controller)
cosign verify --key cosign.pub myregistry/myapp:v1.0.0
```

### Admission Controllers

Kubernetes admission controllers enforce policies before pods are created:
- Only allow signed images
- Block images from untrusted registries
- Enforce security contexts
- Tools: OPA Gatekeeper, Kyverno

---

## 9. Exercises

### Exercise 1: Security Architecture

Design a security architecture for a multi-tenant SaaS e-commerce platform:
1. Authentication strategy (users, services, admins)
2. Authorization model (RBAC vs ABAC, tenant isolation)
3. Secret management approach
4. Network security layers
5. Container security policies

### Exercise 2: Threat Model

Create a threat model (STRIDE) for a microservices payment system:
1. Identify assets, entry points, trust boundaries
2. Identify threats using STRIDE categories
3. Rate risks (likelihood × impact)
4. Propose mitigations for top 5 threats

---

## 10. Self-Check Questions

1. What is zero-trust architecture and how does it differ from perimeter security?
2. Explain the OAuth 2.0 Authorization Code flow.
3. What is mTLS and how does a service mesh make it easier?
4. Why shouldn't you store secrets in environment variables?
5. What container security practices should you enforce?
6. What are Network Policies in Kubernetes?

---

## 11. Answers

### Answer 1
**Perimeter security** trusts everything inside the network — once past the firewall, services communicate freely. **Zero-trust** trusts nothing — every request must be authenticated and authorized regardless of network location. Difference: in perimeter security, a compromised internal service can access everything; in zero-trust, it can only access what its identity is authorized for. Implementation: mTLS between services, JWT validation on every request, network policies, least-privilege service accounts.

### Answer 2
(1) User clicks "login" in the app. (2) App redirects to authorization server's login page. (3) User authenticates with credentials. (4) Auth server redirects back to app with a short-lived authorization code. (5) App's backend exchanges the code for tokens (access_token, refresh_token, id_token) — this exchange happens server-to-server, not through the browser. (6) App uses access_token for API calls. The code exchange on the backend prevents token exposure to the browser.

### Answer 3
**mTLS** = both client and server present certificates and verify each other. Unlike regular TLS where only the server proves identity. Service meshes (Istio, Linkerd) inject sidecar proxies that handle mTLS transparently — automatic certificate issuance, rotation, and validation without changing application code. The app communicates with localhost (the sidecar), and the sidecar handles encryption to the remote sidecar.

### Answer 4
Environment variables are: visible in process listings (`/proc/<pid>/environ`), logged by many frameworks, exposed in error reports/stack traces, inherited by child processes, stored in container runtime metadata. Better alternatives: mounted files (Kubernetes Secrets as volumes), sidecar-injected secrets (Vault Agent), or cloud secret managers accessed via SDK at runtime.

### Answer 5
(1) Run as non-root user. (2) Use minimal base images (distroless/Alpine). (3) Scan images for CVEs. (4) Set `readOnlyRootFilesystem: true`. (5) Drop all capabilities. (6) Disallow privilege escalation. (7) Pin image versions/digests. (8) Use Pod Security Standards (Restricted level).

### Answer 6
Network Policies are Kubernetes firewall rules controlling pod-to-pod traffic. By default, all pods can communicate. Network Policies restrict: ingress (who can call a pod) and egress (what a pod can call). They use label selectors to match pods. A CNI plugin (Calico, Cilium) enforces them. Essential for micro-segmentation and limiting blast radius.

---

*Previous: [08 - Observability & Reliability](08-observability-reliability.md)*  
*Next: [10 - Event-Driven Architecture](10-event-driven-architecture.md)*