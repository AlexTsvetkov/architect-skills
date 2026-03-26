# Prompt: Lesson 11 — CI/CD & GitOps

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 11: CI/CD & GitOps**.

### Context
Course for experienced Java developers. Deep, technical, practical. Java/Spring Boot. GitOps with ArgoCD. Reference: Microsoft DevOps best practices.

### Topic Coverage

1. **CI/CD Pipeline Design**
   - Pipeline stages diagram (Commit→Build→Test→Scan→Publish→Deploy)
   - Microservices CI/CD: independent pipelines per service
   - GitHub Actions complete pipeline YAML (build, test, Docker build, scan, push, deploy)
   - Trunk-based development vs GitFlow comparison table

2. **GitOps Principles**
   - 4 core principles
   - ArgoCD architecture diagram (Git repo → ArgoCD → Kubernetes)
   - Git repository structure for environments (dev/staging/production)
   - Benefits table (audit trail, rollback, consistency, security, review)

3. **Infrastructure as Code**
   - Terraform example (AKS cluster + PostgreSQL)
   - IaC best practices table (5 practices with rationale)

4. **Deployment Strategies** (4 strategies with diagrams)
   - Rolling Update (step-by-step, pros/cons)
   - Blue-Green (dual environment, instant rollback)
   - Canary (gradual traffic shift, 4 phases)
   - A/B Testing (user-attribute-based routing)
   - Comparison table (zero downtime, rollback speed, cost, complexity)

5. **Feature Flags**
   - Decouple deployment from release (flow diagram)
   - Java implementation code
   - Tools table (LaunchDarkly, Unleash, Flagsmith, Split.io)
   - Best practices (remove after rollout, short-lived, track ownership)

6. **Pipeline Security (Shift-Left)**
   - Security types diagram (SAST→SCA→Image Scan→DAST, early→late, cheap→expensive)
   - Tools table (SonarQube, Snyk, Trivy, OWASP ZAP)

### Code Examples Required
1. Complete GitHub Actions pipeline YAML
2. Terraform HCL for K8s cluster
3. ArgoCD Application YAML
4. Feature flag implementation (Java if/else with flag service)
5. Trivy image scan command in CI

### Diagrams Required (minimum 7)
CI/CD pipeline stages, ArgoCD architecture, GitOps repository structure, Rolling update, Blue-green, Canary deployment phases, Shift-left security spectrum

### Real-World Case Study
Design CI/CD strategy for 8-service e-commerce platform: pipeline stages, branching, promotion flow, rollback, security scanning, GitOps repo structure.

### Exercises (3)
1. CI/CD pipeline design for 8 microservices (stages, branching, promotion, rollback, security)
2. Deployment strategy for payment service API change (strategy choice, DB migration, monitoring, rollback)
3. GitOps repository structure (8 services, 3 environments, shared infra, per-team config)

### Self-Check Questions (6)
Cover: CI/CD stages, GitOps principles, deployment strategies comparison, feature flags, shift-left security, trunk-based vs GitFlow.

### References
Include: "The Phoenix Project" (Gene Kim), "Continuous Delivery" (Humble & Farley), ArgoCD docs, GitHub Actions docs, Microsoft DevOps resource center.