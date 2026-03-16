# Lesson 11: CI/CD & GitOps

> **Prerequisites:** Lessons 01-10  
> **Time estimate:** 6-8 hours  
> **Parallel reading:** "The Phoenix Project" — Gene Kim

---

## Table of Contents

1. [CI/CD Pipeline Design](#1-cicd-pipeline-design)
2. [GitOps Principles](#2-gitops-principles)
3. [Infrastructure as Code](#3-infrastructure-as-code)
4. [Deployment Strategies](#4-deployment-strategies)
5. [Feature Flags](#5-feature-flags)
6. [Pipeline Security](#6-pipeline-security)
7. [Exercises](#7-exercises)
8. [Self-Check Questions](#8-self-check-questions)
9. [Answers](#9-answers)

---

## 1. CI/CD Pipeline Design

### Pipeline Stages

```
┌────────┐  ┌─────────┐  ┌──────────┐  ┌─────────┐  ┌──────────┐  ┌────────┐
│ Commit │→ │  Build  │→ │   Test   │→ │  Scan   │→ │  Publish │→ │ Deploy │
└────────┘  └─────────┘  └──────────┘  └─────────┘  └──────────┘  └────────┘

Commit:  Push to Git
Build:   Compile, package (mvn clean package)
Test:    Unit, integration, contract tests
Scan:    SAST, dependency check, image scan
Publish: Push Docker image to registry
Deploy:  Kubernetes rollout (dev → staging → prod)
```

### Microservices CI/CD

```
Each service has its OWN pipeline:
  order-service repo → order-service pipeline → order-service deployment
  payment-service repo → payment-service pipeline → payment-service deployment

Independent pipelines enable:
  - Independent deployments
  - Different release cadences
  - Team autonomy
```

### Example: GitHub Actions Pipeline

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with: { java-version: '21', distribution: 'temurin' }

    - name: Build and Test
      run: mvn clean verify

    - name: Build Docker Image
      run: docker build -t myregistry/order-service:${{ github.sha }} .

    - name: Scan Image
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: myregistry/order-service:${{ github.sha }}
        severity: 'CRITICAL,HIGH'
        exit-code: '1'

    - name: Push Image
      run: |
        docker push myregistry/order-service:${{ github.sha }}
        docker tag myregistry/order-service:${{ github.sha }} myregistry/order-service:latest
        docker push myregistry/order-service:latest

  deploy-staging:
    needs: build-and-test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - name: Update Kubernetes manifest
      run: |
        # Update image tag in GitOps repo
        # ArgoCD will detect the change and deploy
```

### Trunk-Based Development vs. GitFlow

| Aspect | Trunk-Based | GitFlow |
|--------|------------|---------|
| Branches | Short-lived feature branches, merge to main quickly | Long-lived develop, release, hotfix branches |
| Release | Any commit on main is deployable | Explicit release branches |
| Merge conflicts | Rare (short branches) | Common (long branches) |
| CI/CD fit | Excellent (continuous integration) | Poor (integration happens late) |
| Best for | Microservices, frequent deployments | Packaged software with versioned releases |

**Recommendation for microservices:** Trunk-based development with feature flags.

---

## 2. GitOps Principles

### Core Principles

```
1. Git is the single source of truth for desired state
2. All changes are made through Git (pull requests)
3. Agents reconcile actual state → desired state automatically
4. No kubectl apply from laptops!

Developer → Git PR → Merge → GitOps Agent → Kubernetes
                                (ArgoCD/Flux)
```

### ArgoCD Architecture

```
┌────────────────────────────────────────────────┐
│                  GIT REPOSITORY                 │
│                                                 │
│  environments/                                  │
│  ├── dev/                                       │
│  │   ├── order-service.yaml  (image: v1.2.3)  │
│  │   └── payment-service.yaml                   │
│  ├── staging/                                   │
│  │   ├── order-service.yaml  (image: v1.2.2)  │
│  │   └── payment-service.yaml                   │
│  └── production/                                │
│      ├── order-service.yaml  (image: v1.2.1)  │
│      └── payment-service.yaml                   │
└─────────────────────┬──────────────────────────┘
                      │ watches
                ┌─────┴──────┐
                │   ArgoCD   │
                │            │
                │ Detects    │
                │ drift      │
                │ Auto-sync  │
                └─────┬──────┘
                      │ applies
                ┌─────┴──────┐
                │ Kubernetes │
                │  Cluster   │
                └────────────┘
```

### Benefits of GitOps

| Benefit | Description |
|---------|-------------|
| Audit trail | Every change is a Git commit with author and timestamp |
| Rollback | `git revert` restores previous state |
| Consistency | Cluster state always matches Git |
| Security | No direct cluster access needed; changes through PRs |
| Review | Pull requests for infrastructure changes |

---

## 3. Infrastructure as Code

### Terraform (Multi-Cloud)

```hcl
# main.tf — Provision Kubernetes cluster
resource "azurerm_kubernetes_cluster" "main" {
  name                = "ecommerce-aks"
  location            = "westeurope"
  resource_group_name = azurerm_resource_group.main.name
  dns_prefix          = "ecommerce"

  default_node_pool {
    name       = "default"
    node_count = 3
    vm_size    = "Standard_D4s_v3"
  }

  identity {
    type = "SystemAssigned"
  }
}

resource "azurerm_postgresql_flexible_server" "main" {
  name                = "ecommerce-db"
  location            = "westeurope"
  resource_group_name = azurerm_resource_group.main.name
  sku_name            = "GP_Standard_D2s_v3"
  version             = "16"
}
```

```bash
terraform plan    # Preview changes
terraform apply   # Apply changes
terraform destroy # Tear down
```

### IaC Best Practices

| Practice | Why |
|----------|-----|
| Version control all IaC | Audit trail, rollback capability |
| Use modules for reuse | DRY, consistent patterns |
| State management (remote) | Shared state for team collaboration |
| Plan before apply | Review changes before execution |
| Immutable infrastructure | Replace, don't patch |

---

## 4. Deployment Strategies

### Rolling Update (Default in Kubernetes)

```
v1 v1 v1 v1    (start: 4 replicas on v1)
v1 v1 v1 v2    (replace one at a time)
v1 v1 v2 v2
v1 v2 v2 v2
v2 v2 v2 v2    (end: all on v2)

Pros: Zero downtime, simple, built into Kubernetes
Cons: Both versions run simultaneously during rollout
      Rollback takes time (roll forward to v1)
```

### Blue-Green Deployment

```
Blue (current):   [v1] [v1] [v1]  ← serving traffic
Green (new):      [v2] [v2] [v2]  ← ready, not serving

Test green → switch traffic → green serves
Blue kept as instant rollback

Pros: Instant rollback (switch back to blue)
Cons: Double infrastructure cost during deployment
```

### Canary Deployment

```
Phase 1:  [v1] [v1] [v1] [v1] [v2]    (5% traffic to v2)
          Monitor metrics...
Phase 2:  [v1] [v1] [v1] [v2] [v2]    (25% traffic to v2)
          Monitor metrics...
Phase 3:  [v1] [v2] [v2] [v2] [v2]    (75% traffic to v2)
          Monitor metrics...
Phase 4:  [v2] [v2] [v2] [v2] [v2]    (100% traffic to v2)

Pros: Gradual rollout, catch issues early, data-driven
Cons: Complex traffic management, longer rollout
Tools: Istio, Flagger, Argo Rollouts
```

### A/B Testing

```
Route based on user attributes (not random):
  Users in group A → v1 (control)
  Users in group B → v2 (experiment)

Measure business metrics (conversion, engagement)
Use for product decisions, not just deployments
```

### Comparison

| Strategy | Zero Downtime | Rollback Speed | Cost | Complexity |
|----------|:-----:|:-----------:|:----:|:----------:|
| Rolling | ✅ | Medium | Low | Low |
| Blue-Green | ✅ | Instant | High | Medium |
| Canary | ✅ | Fast | Medium | High |
| A/B Test | ✅ | Fast | Medium | High |

---

## 5. Feature Flags

### Why Feature Flags

Decouple deployment from release:
```
Deploy code to production → feature is OFF
Enable for internal testers → feature is ON for 5 users
Enable for 10% of users → gradual rollout
Enable for all → full release
Disable instantly if problems → kill switch
```

### Implementation

```java
// Feature flag check
if (featureFlags.isEnabled("new-checkout-flow", userId)) {
    return newCheckoutService.process(order);
} else {
    return legacyCheckoutService.process(order);
}
```

### Feature Flag Tools

| Tool | Type |
|------|------|
| LaunchDarkly | SaaS (most popular) |
| Unleash | Open-source |
| Flagsmith | Open-source + SaaS |
| Split.io | SaaS |

### Best Practices

- Remove flags after full rollout (avoid flag debt)
- Use flags for short-lived features (not permanent configuration)
- Track flag usage and ownership
- Default to "off" for new features

---

## 6. Pipeline Security

### Shift-Left Security

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│   SAST   │  │   SCA    │  │  Image   │  │   DAST   │
│ (Static  │  │(Software │  │  Scan    │  │(Dynamic  │
│  Analysis)│  │Composition│  │(Container│  │ Analysis)│
│          │  │ Analysis)│  │  CVEs)   │  │          │
│ In IDE + │  │ In build │  │ Before   │  │ In       │
│ CI       │  │          │  │ deploy   │  │ staging  │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
   Early ←────────────────────────────────────→ Late
   Cheap ←────────────────────────────────────→ Expensive
```

| Type | What | Tools |
|------|------|-------|
| SAST | Find bugs in source code | SonarQube, Semgrep, CodeQL |
| SCA | Find vulnerable dependencies | Snyk, Dependabot, OWASP Dep-Check |
| Image Scan | Find CVEs in container images | Trivy, Grype, Clair |
| DAST | Find vulnerabilities in running app | OWASP ZAP, Burp Suite |

---

## 7. Exercises

### Exercise 1: CI/CD Pipeline Design

Design a CI/CD pipeline for a microservices platform with 8 services:
1. Pipeline stages for each service
2. Branching strategy
3. Promotion flow (dev → staging → production)
4. Rollback procedure
5. Security scanning integration

### Exercise 2: Deployment Strategy

You're deploying a new version of the payment service that changes the payment API. Design:
1. Which deployment strategy would you use and why?
2. How would you handle database schema changes?
3. How would you monitor the deployment?
4. What's your rollback plan?

### Exercise 3: GitOps Setup

Design a GitOps repository structure for:
- 8 microservices
- 3 environments (dev, staging, production)
- Shared infrastructure (Kafka, Redis, PostgreSQL)
- Per-team configuration

---

## 8. Self-Check Questions

1. What are the stages of a typical CI/CD pipeline for microservices?
2. Explain GitOps and its core principles.
3. Compare rolling, blue-green, and canary deployment strategies.
4. What are feature flags and how do they decouple deployment from release?
5. What is "shift-left" security in CI/CD?
6. When would you choose trunk-based development over GitFlow?

---

## 9. Answers

### Answer 1
Typical stages: (1) **Commit** — code pushed to Git. (2) **Build** — compile and package (`mvn clean package`). (3) **Unit/Integration Test** — run automated tests. (4) **Security Scan** — SAST, SCA, image scanning. (5) **Publish** — push Docker image to container registry with version tag. (6) **Deploy to Dev** — automatic deployment to dev environment. (7) **Deploy to Staging** — automatic or manual gate. (8) **Deploy to Production** — manual approval or automated canary.

### Answer 2
**GitOps:** Infrastructure and application state stored in Git. An agent (ArgoCD, Flux) watches the Git repo and continuously reconciles the Kubernetes cluster to match the desired state in Git. Core principles: (1) Git is the single source of truth. (2) All changes via pull requests (auditable, reviewable). (3) Automated reconciliation (agent syncs cluster to Git). (4) Self-healing (drift is detected and corrected).

### Answer 3
**Rolling:** Replace instances one at a time. Simple, zero downtime, built into Kubernetes. Slow rollback. **Blue-Green:** Two complete environments; switch traffic atomically. Instant rollback. Double cost during deployment. **Canary:** Route small % of traffic to new version, gradually increase. Catch issues early with minimal user impact. Most complex to implement (needs traffic management).

### Answer 4
Feature flags are conditional code paths controlled by external configuration. They decouple deployment (code in production) from release (feature active for users). Benefits: deploy dark (code live but feature off), gradual rollout (enable for 5% → 50% → 100%), instant kill switch (disable without redeployment), A/B testing (different feature variants for different users), trunk-based development (incomplete features behind flags).

### Answer 5
Shift-left means moving security testing earlier in the development lifecycle — from production (right) to development (left). Earlier detection = cheaper fixes. Implementation: SAST in IDE and CI (catch code issues), SCA in build (catch vulnerable dependencies), container scanning before deploy (catch image CVEs), DAST in staging (catch runtime vulnerabilities). The goal is to catch 80% of security issues before code reaches production.

### Answer 6
Choose trunk-based for microservices with frequent deployments, continuous delivery, small teams owning individual services, where feature flags replace long branches. Choose GitFlow for software with versioned releases (libraries, SDKs), when multiple release versions must be maintained simultaneously, or when the team isn't ready for continuous deployment.

---

*Previous: [10 - Event-Driven Architecture](10-event-driven-architecture.md)*  
*Next: [12 - Scalability & Performance](12-scalability-performance.md)*