# Prompt: Lesson 09 — Security in Cloud-Native Systems

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 09: Security in Cloud-Native Systems**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot with Spring Security. Reference: Microsoft Zero Trust Architecture, OWASP.

### Topic Coverage

1. **Zero-Trust Architecture**
   - Traditional perimeter security vs zero-trust (visual comparison)
   - Zero-trust pillars table (verify identity, least privilege, assume breach, verify explicitly, micro-segmentation)

2. **Identity and Access Management**
   - Authentication vs Authorization
   - RBAC vs ABAC vs ReBAC comparison table

3. **OAuth 2.0 / OpenID Connect Deep Dive**
   - Authorization Code Flow (6-step with diagram)
   - Client Credentials Flow (service-to-service)
   - JWT validation steps (6 checks, stateless)
   - Token architecture in microservices (Gateway → Service → Service propagation)

4. **Service-to-Service Authentication**
   - mTLS (mutual TLS) — how it works, service mesh automation
   - JWT Propagation — user context flowing through service chain

5. **Secret Management**
   - What NOT to do (4 anti-patterns)
   - Kubernetes Secrets (minimum, base64 ≠ encryption)
   - HashiCorp Vault (architecture with sidecar agent, benefits: dynamic secrets, rotation, audit)
   - Cloud-native options table (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager)

6. **Container Security**
   - Image security (5 practices)
   - Runtime security (Pod Security Context YAML with all hardening)
   - Pod Security Standards table (Privileged, Baseline, Restricted)

7. **Network Security**
   - Network Policies (default deny YAML + allow specific traffic YAML)
   - Defense in Depth (5 layers: cloud firewall → K8s network policies → service mesh mTLS → JWT → RBAC)

8. **Supply Chain Security**
   - SBOM concept
   - Image signing with cosign
   - Admission controllers (OPA Gatekeeper, Kyverno)

### Code Examples Required
1. OAuth 2.0 Authorization Code flow implementation (Spring Security)
2. JWT validation configuration (Spring Security)
3. Pod Security Context YAML (complete hardening)
4. Network Policy YAML (default deny + specific allow)
5. Kubernetes Secret YAML
6. Image signing commands (cosign)

### Diagrams Required (minimum 6)
Zero-trust vs perimeter, OAuth 2.0 Authorization Code flow, Token architecture (Gateway→Service→Service), mTLS with service mesh sidecars, Vault with sidecar agent, Defense in depth layers

### Real-World Case Study
Design security architecture for a multi-tenant SaaS e-commerce platform: authentication (users, services, admins), authorization (RBAC + tenant isolation), secrets, network security, container security.

### Exercises (2)
1. Security architecture design (5 aspects for multi-tenant SaaS)
2. STRIDE threat model for payment system

### Self-Check Questions (6)
Cover: zero-trust vs perimeter, OAuth 2.0 Authorization Code flow, mTLS + service mesh, secrets in env vars, container security, Network Policies.

### References
Include: OWASP, Microsoft Zero Trust documentation, Spring Security docs, HashiCorp Vault docs, Kubernetes security best practices.