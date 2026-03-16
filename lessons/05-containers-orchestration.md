# Lesson 05: Containers & Orchestration (Docker / Kubernetes)

> **Prerequisites:** Lessons 01-04  
> **Time estimate:** 8-10 hours  
> **Parallel reading:** "Kubernetes in Action" (2nd Ed) — Marko Lukša

---

## Table of Contents

1. [Docker Deep Dive](#1-docker-deep-dive)
2. [Kubernetes Architecture](#2-kubernetes-architecture)
3. [Kubernetes Core Objects](#3-kubernetes-core-objects)
4. [Advanced Workloads](#4-advanced-workloads)
5. [Kubernetes Networking](#5-kubernetes-networking)
6. [Persistent Storage](#6-persistent-storage)
7. [Resource Management](#7-resource-management)
8. [Helm Charts](#8-helm-charts)
9. [Managed Kubernetes: AKS vs EKS vs GKE](#9-managed-kubernetes)
10. [Exercises](#10-exercises)
11. [Self-Check Questions](#11-self-check-questions)
12. [Answers](#12-answers)

---

## 1. Docker Deep Dive

### Images, Layers, and Caching

```
Docker Image (layered):
┌────────────────────────┐
│ Layer 5: COPY app.jar  │  ← Changes often → rebuild this layer
├────────────────────────┤
│ Layer 4: RUN mvn build │
├────────────────────────┤
│ Layer 3: COPY pom.xml  │  ← Dependencies (cached if unchanged)
├────────────────────────┤
│ Layer 2: RUN apt-get   │
├────────────────────────┤
│ Layer 1: eclipse-temurin│  ← Base image (cached)
└────────────────────────┘

Rule: Order from least-changing → most-changing
```

### Multi-Stage Build (Essential for Java)

```dockerfile
# Stage 1: BUILD
FROM eclipse-temurin:21-jdk AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: RUNTIME (smaller image)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/app.jar .
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 8080
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Result: Build image ~800MB → Runtime image ~200MB.

### Security Best Practices

| Practice | Why |
|----------|-----|
| Run as non-root user | Limit container escape blast radius |
| Use minimal base images (Alpine, distroless) | Fewer CVEs |
| Scan images (`trivy image myapp:latest`) | Catch known vulnerabilities |
| Pin base image versions | Reproducible builds |
| Don't store secrets in images | Use runtime env vars / mounted secrets |
| Use `.dockerignore` | Exclude `.git`, `target/`, IDE files |

### Docker Compose for Local Dev

```yaml
services:
  app:
    build: .
    ports: ["8080:8080"]
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/myapp
      SPRING_REDIS_HOST: redis
    depends_on:
      db: { condition: service_healthy }

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
    volumes: [pgdata:/var/lib/postgresql/data]
    healthcheck:
      test: pg_isready -U postgres
      interval: 5s

  redis:
    image: redis:7-alpine

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    ports: ["9092:9092"]

volumes:
  pgdata:
```

---

## 2. Kubernetes Architecture

### Control Plane + Worker Nodes

```
┌─────────────────────── CONTROL PLANE ───────────────────────┐
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│  │ API Server   │  │  Scheduler   │  │ Controller Manager│  │
│  │ (kube-apisvr)│  │ (assigns pods│  │ (ensures desired  │  │
│  │              │  │  to nodes)   │  │  state = actual)  │  │
│  └──────┬───────┘  └──────────────┘  └───────────────────┘  │
│         │                                                    │
│  ┌──────┴───────┐                                            │
│  │    etcd      │  (distributed key-value store)             │
│  │  (cluster    │  (single source of truth for all state)    │
│  │   state)     │                                            │
│  └──────────────┘                                            │
└──────────────────────────────────────────────────────────────┘
              │
              │  kubelet communicates with API server
              │
┌─────────────┴──── WORKER NODE 1 ────────────────────────────┐
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────────┐   │
│  │ kubelet  │  │kube-proxy│  │  Container Runtime       │   │
│  │ (node    │  │ (network │  │  (containerd / CRI-O)    │   │
│  │  agent)  │  │  rules)  │  │                          │   │
│  └──────────┘  └──────────┘  └──────────────────────────┘   │
│                                                              │
│  ┌──────────────────┐  ┌──────────────────┐                  │
│  │   Pod A          │  │   Pod B          │                  │
│  │ ┌──────────────┐ │  │ ┌──────────────┐ │                  │
│  │ │ Container 1  │ │  │ │ Container 1  │ │                  │
│  │ └──────────────┘ │  │ └──────────────┘ │                  │
│  └──────────────────┘  └──────────────────┘                  │
└──────────────────────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Pod** | Smallest deployable unit. One or more containers sharing network/storage. |
| **ReplicaSet** | Ensures a specified number of pod replicas are running. |
| **Deployment** | Manages ReplicaSets. Handles rolling updates and rollbacks. |
| **Service** | Stable network endpoint for a set of pods (load balancing). |
| **Namespace** | Virtual cluster within a cluster for isolation. |

### Declarative vs. Imperative

```
Imperative (avoid):
  kubectl run nginx --image=nginx
  kubectl scale deployment nginx --replicas=3

Declarative (preferred):
  kubectl apply -f deployment.yaml

  The YAML describes desired state.
  Kubernetes continuously reconciles actual state → desired state.
  This is GitOps-friendly (version control your YAML).
```

---

## 3. Kubernetes Core Objects

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: ecommerce
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above desired during update
      maxUnavailable: 0   # Zero downtime
  template:
    metadata:
      labels:
        app: order-service
        version: v1.2.3
    spec:
      containers:
      - name: order-service
        image: myregistry/order-service:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - port: 80           # Service port
    targetPort: 8080    # Container port
  type: ClusterIP       # Internal only (default)

# Other services call: http://order-service/api/orders
# Kubernetes DNS resolves the name to pod IPs
```

### Service Types

| Type | Description | Use Case |
|------|-------------|----------|
| **ClusterIP** | Internal only. Default. | Service-to-service communication |
| **NodePort** | Exposes on each node's IP at a static port | Development, testing |
| **LoadBalancer** | Provisions cloud load balancer | External-facing services |
| **ExternalName** | CNAME record to external service | Access external DB from cluster |

### ConfigMap and Secret

```yaml
# ConfigMap — non-sensitive config
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  SPRING_PROFILES_ACTIVE: "production"
  LOG_LEVEL: "INFO"
  KAFKA_BOOTSTRAP_SERVERS: "kafka:9092"

---
# Secret — sensitive data (base64 encoded, encrypted at rest)
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  url: amRiYzpwb3N0Z3Jlc3FsOi8vZGI6NTQzMi9teWFwcA==   # base64
  username: YWRtaW4=
  password: c2VjcmV0
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  tls:
  - hosts: [api.myshop.com]
    secretName: tls-secret
  rules:
  - host: api.myshop.com
    http:
      paths:
      - path: /orders
        pathType: Prefix
        backend:
          service: { name: order-service, port: { number: 80 } }
      - path: /products
        pathType: Prefix
        backend:
          service: { name: product-service, port: { number: 80 } }
      - path: /payments
        pathType: Prefix
        backend:
          service: { name: payment-service, port: { number: 80 } }
```

---

## 4. Advanced Workloads

### StatefulSet (for stateful apps)

```
Deployment (stateless):     StatefulSet (stateful):
  pod-abc123                  pod-0
  pod-def456                  pod-1
  pod-ghi789                  pod-2

StatefulSet provides:
- Stable network identities (pod-0, pod-1, pod-2)
- Ordered deployment and scaling (0 → 1 → 2)
- Persistent volume per pod
- Ordered termination (2 → 1 → 0)

Use for: Databases, Kafka, Elasticsearch, ZooKeeper
```

### DaemonSet (one pod per node)

```
Use for: Log collectors, monitoring agents, network plugins
Every node gets exactly one pod.

Example: Fluentd log collector on every node
```

### Job and CronJob

```yaml
# Job — run once to completion
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
        command: ["java", "-jar", "app.jar", "--migrate"]
      restartPolicy: Never
  backoffLimit: 3

---
# CronJob — run on schedule
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-report
spec:
  schedule: "0 2 * * *"    # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: reporter
            image: myapp:v1.2.3
            command: ["java", "-jar", "app.jar", "--report"]
          restartPolicy: Never
```

---

## 5. Kubernetes Networking

### Pod-to-Pod Communication

```
Every pod gets its own IP address.
All pods can communicate with all other pods without NAT.
All nodes can communicate with all pods without NAT.

Pod A (10.244.1.5) → Pod B (10.244.2.10): direct IP communication
```

### Network Policies (Firewall Rules)

```yaml
# Only allow traffic from order-service to payment-service
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payment-service-policy
spec:
  podSelector:
    matchLabels:
      app: payment-service
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: order-service
    ports:
    - port: 8080
```

### Service Mesh (Istio/Linkerd)

```
Without service mesh:
  App A ──HTTP──→ App B

With service mesh (sidecar proxy):
  ┌───────────┐        ┌───────────┐
  │  App A    │        │  App B    │
  │           │        │           │
  │ ┌───────┐ │  mTLS  │ ┌───────┐ │
  │ │Envoy  │─┼────────┼→│Envoy  │ │
  │ │Proxy  │ │        │ │Proxy  │ │
  │ └───────┘ │        │ └───────┘ │
  └───────────┘        └───────────┘

Service mesh provides (without changing app code):
- mTLS encryption between services
- Traffic management (canary, circuit breaking)
- Observability (metrics, tracing)
- Access control
```

---

## 6. Persistent Storage

```
┌──────────────┐     claims     ┌──────────────┐    binds    ┌──────────┐
│     Pod      │ ──────────────→│     PVC      │ ──────────→ │    PV    │
│              │                │ (Persistent  │             │(Persistent│
│ volumeMount: │                │  Volume Claim)│             │  Volume) │
│  /data       │                │              │             │          │
└──────────────┘                │ 10Gi, RWO    │             │ AWS EBS  │
                                └──────────────┘             └──────────┘
```

```yaml
# PVC — request for storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: gp3          # AWS EBS gp3
  resources:
    requests:
      storage: 50Gi
```

### Access Modes

| Mode | Abbreviation | Description |
|------|-------------|-------------|
| ReadWriteOnce | RWO | Single node read/write |
| ReadOnlyMany | ROX | Multiple nodes read-only |
| ReadWriteMany | RWX | Multiple nodes read/write (NFS, EFS) |

---

## 7. Resource Management

### Requests vs. Limits

```yaml
resources:
  requests:          # Guaranteed minimum
    memory: "256Mi"  # Scheduler uses this to place pods
    cpu: "250m"      # 250 millicores = 0.25 CPU
  limits:            # Maximum allowed
    memory: "512Mi"  # Pod killed (OOMKilled) if exceeded
    cpu: "500m"      # Pod throttled if exceeded
```

### QoS Classes

| Class | Condition | Behavior |
|-------|-----------|----------|
| **Guaranteed** | requests = limits for all containers | Last to be evicted |
| **Burstable** | requests < limits | Evicted after BestEffort |
| **BestEffort** | No requests or limits set | First to be evicted |

**Recommendation:** Always set requests. Set memory limits. CPU limits are debatable (throttling can cause latency spikes).

### Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

---

## 8. Helm Charts

### What is Helm?

Package manager for Kubernetes. A Helm chart is a bundle of Kubernetes YAML templates with configurable values.

```
mychart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── templates/
│   ├── deployment.yaml # Templated K8s manifests
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl    # Template helpers
└── charts/             # Sub-chart dependencies
```

### Templated Deployment

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

```yaml
# values.yaml (defaults)
replicaCount: 2
image:
  repository: myregistry/order-service
  tag: "v1.0.0"
resources:
  requests: { memory: "256Mi", cpu: "250m" }
  limits: { memory: "512Mi", cpu: "500m" }
```

```bash
# Install/upgrade
helm install order-service ./mychart --namespace ecommerce
helm upgrade order-service ./mychart --set image.tag=v1.1.0
helm rollback order-service 1  # rollback to revision 1
```

---

## 9. Managed Kubernetes

| Feature | AKS (Azure) | EKS (AWS) | GKE (Google) |
|---------|-------------|-----------|--------------|
| Control plane cost | Free | ~$73/month | Free (Autopilot) |
| Easiest setup | ★★★★ | ★★★ | ★★★★★ |
| Best integration | Azure services | AWS services | GCP services |
| Auto-scaling | KEDA, cluster autoscaler | Karpenter, cluster autoscaler | Autopilot (fully managed) |
| Service mesh | Istio add-on | App Mesh | Istio managed |
| Unique feature | Azure AD integration | Fargate (serverless pods) | Autopilot mode |

**Recommendation for learning:** Start with a local cluster (minikube, kind, or Docker Desktop) then move to a managed service.

---

## 10. Exercises

### Exercise 1: Containerize a Spring Boot App

1. Create a multi-stage Dockerfile for a Spring Boot application
2. Ensure it runs as non-root user
3. Add a health check
4. Create a docker-compose.yml with PostgreSQL and Redis
5. Verify the image size is under 300MB

### Exercise 2: Deploy to Kubernetes

Create Kubernetes manifests for a microservice:
1. Deployment with 3 replicas, rolling update, readiness/liveness probes
2. Service (ClusterIP)
3. ConfigMap for application configuration
4. Secret for database credentials
5. Ingress for external access
6. HPA for auto-scaling at 70% CPU
7. NetworkPolicy to restrict access

### Exercise 3: Helm Chart

Convert the manifests from Exercise 2 into a Helm chart with configurable values for: replicas, image tag, resource limits, and environment-specific config.

### Exercise 4: Architecture Design

Design a Kubernetes deployment architecture for an e-commerce platform with:
- 8 microservices
- PostgreSQL, Redis, Kafka
- Namespace strategy
- Resource quotas per team
- Network policies
- Ingress routing

---

## 11. Self-Check Questions

1. What is a multi-stage Docker build and why is it important for Java applications?
2. Explain the difference between a Deployment and a StatefulSet. When would you use each?
3. What are readiness and liveness probes? What happens if each fails?
4. Explain Kubernetes Service types: ClusterIP, NodePort, LoadBalancer.
5. What is the difference between resource requests and limits?
6. How does Kubernetes service discovery work? Why don't you need Eureka?
7. What is a Network Policy and why is it important?
8. When would you use a DaemonSet?

---

## 12. Answers

### Answer 1
A multi-stage build uses multiple `FROM` statements. The first stage has build tools (JDK, Maven) and compiles the application. The second stage has only the runtime (JRE). The final image contains only the JAR and JRE, not the build tools. For Java, this reduces image size from ~800MB to ~200MB and eliminates build-time security vulnerabilities from the runtime image.

### Answer 2
**Deployment:** For stateless applications. Pods have random names, no stable identity, no persistent storage per pod. Pods are interchangeable. Use for: web services, APIs, workers.

**StatefulSet:** For stateful applications. Pods have stable names (pod-0, pod-1), stable network identity, ordered deployment/scaling, and persistent volume per pod. Use for: databases, Kafka brokers, Elasticsearch nodes, ZooKeeper.

### Answer 3
**Readiness probe:** Is the pod ready to receive traffic? If it fails, Kubernetes removes the pod from Service endpoints (stops sending traffic). The pod is NOT restarted. Use case: waiting for DB connection, warming caches.

**Liveness probe:** Is the pod alive and functioning? If it fails, Kubernetes restarts the pod. Use case: detecting deadlocks, unrecoverable states.

**Best practice for Spring Boot:** Use `/actuator/health/readiness` and `/actuator/health/liveness` (separate endpoints in Spring Boot 2.3+).

### Answer 4
- **ClusterIP:** Internal only. Gets a cluster-internal IP. Other pods reach it by service name via DNS. Default type. Use for all service-to-service communication.
- **NodePort:** Extends ClusterIP. Also opens a port (30000-32767) on every node. External clients can access via `<NodeIP>:<NodePort>`. Use for development/testing.
- **LoadBalancer:** Extends NodePort. Also provisions a cloud load balancer (AWS ALB, Azure LB). External clients access via the load balancer's public IP. Use for production external-facing services.

### Answer 5
**Requests:** Guaranteed minimum resources. The scheduler uses requests to decide which node to place the pod on. The pod is guaranteed these resources.

**Limits:** Maximum resources allowed. For memory: exceeding the limit causes OOMKill. For CPU: exceeding causes throttling (pod runs slower, not killed).

**Best practice:** Always set requests (for scheduling). Set memory limits (prevent runaway memory). CPU limits are debatable — throttling causes latency spikes; some teams omit CPU limits.

### Answer 6
Kubernetes provides built-in service discovery through DNS. When you create a Service named `order-service` in namespace `ecommerce`, Kubernetes DNS creates an entry: `order-service.ecommerce.svc.cluster.local`. Any pod can reach it at `http://order-service` (within the same namespace) or the FQDN. The Service load-balances across matching pods. You don't need Eureka because Kubernetes handles registration (pods register automatically), discovery (DNS), and load balancing (kube-proxy).

### Answer 7
A NetworkPolicy is a Kubernetes firewall rule that controls which pods can communicate with which. By default, all pods can talk to all pods (no isolation). NetworkPolicies enable:
- Restricting ingress (who can call this service)
- Restricting egress (what this service can call)
- Namespace isolation
Important for: security, compliance, blast radius reduction (compromised pod can't reach everything).

### Answer 8
DaemonSet ensures exactly one pod runs on every node (or selected nodes). Use cases: log collectors (Fluentd, Filebeat), monitoring agents (Prometheus node exporter), network plugins (Cilium, Calico), storage daemons. When a new node joins the cluster, the DaemonSet automatically schedules a pod on it.

---

*Previous: [04 - Distributed Systems & Data](04-distributed-systems-data.md)*  
*Next: [06 - Cloud-Native Design Patterns](06-cloud-native-patterns.md)*