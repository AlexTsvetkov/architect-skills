# Prompt: Lesson 05 — Containers & Orchestration

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 05: Containers & Orchestration (Docker / Kubernetes)**.

### Context
Course for experienced Java developers learning Kubernetes from scratch. Deep, technical, practical. Java/Spring Boot stack.

### Topic Coverage

1. **Docker Deep Dive**
   - Images, layers, caching (layer diagram with ordering rule)
   - Multi-stage build for Java (complete Dockerfile: JDK build → JRE runtime, 800MB → 200MB)
   - Security best practices table (6 practices with rationale)
   - Docker Compose for local dev (complete YAML: app, PostgreSQL, Redis, Kafka with health checks)

2. **Kubernetes Architecture**
   - Control Plane diagram (API Server, Scheduler, Controller Manager, etcd)
   - Worker Node diagram (kubelet, kube-proxy, container runtime, pods)
   - Key concepts table (Pod, ReplicaSet, Deployment, Service, Namespace)
   - Declarative vs Imperative (with examples, GitOps connection)

3. **Kubernetes Core Objects** (with complete YAML examples)
   - Deployment (rolling update strategy, probes, resource limits, env from secrets)
   - Service (ClusterIP, NodePort, LoadBalancer, ExternalName — table)
   - ConfigMap and Secret
   - Ingress (with TLS, path-based routing)

4. **Advanced Workloads**
   - StatefulSet (stable identities, ordered deployment, use cases)
   - DaemonSet (one per node, use cases)
   - Job and CronJob (YAML examples)

5. **Kubernetes Networking**
   - Pod-to-pod communication model
   - Network Policies (YAML example: restrict payment-service access)
   - Service Mesh (Istio/Linkerd) with sidecar proxy diagram

6. **Persistent Storage**
   - PV → PVC → Pod chain diagram
   - PVC YAML example
   - Access modes table (RWO, ROX, RWX)

7. **Resource Management**
   - Requests vs Limits (YAML, explanation of scheduling vs throttling/OOMKill)
   - QoS Classes table (Guaranteed, Burstable, BestEffort)
   - HPA YAML example (CPU + memory metrics)

8. **Helm Charts**
   - Chart structure
   - Templated deployment example
   - values.yaml example
   - Install/upgrade/rollback commands

9. **Managed Kubernetes: AKS vs EKS vs GKE**
   - Comparison table (cost, setup, integration, auto-scaling, service mesh, unique features)

### Code Examples Required
1. Multi-stage Dockerfile for Spring Boot (complete with security best practices)
2. Docker Compose for full development environment
3. Complete Kubernetes Deployment YAML with probes, resources, secrets
4. Service + Ingress YAML
5. HPA YAML
6. Network Policy YAML
7. Helm chart template + values

### Diagrams Required (minimum 6)
Docker image layers, K8s architecture (control plane + worker), Pod-to-Service networking, PV/PVC chain, Service mesh sidecar, StatefulSet vs Deployment

### Exercises (4)
1. Containerize Spring Boot app (Dockerfile, docker-compose, <300MB image)
2. Deploy to Kubernetes (7 manifests)
3. Create Helm chart
4. Design K8s architecture for e-commerce platform (8 services)

### Self-Check Questions (8)
Cover: multi-stage builds, Deployment vs StatefulSet, readiness/liveness probes, Service types, requests vs limits, K8s service discovery, Network Policies, DaemonSet.