# Lesson 14: Cost Optimization & FinOps

> **Prerequisites:** Lessons 01-13  
> **Time estimate:** 4-6 hours

---

## Table of Contents

1. [Cloud Cost Model](#1-cloud-cost-model)
2. [Right-Sizing](#2-right-sizing)
3. [Spot/Preemptible Instances](#3-spot-instances)
4. [Serverless Economics](#4-serverless-economics)
5. [FinOps Practices](#5-finops-practices)
6. [Exercises](#6-exercises)
7. [Self-Check Questions](#7-self-check-questions)
8. [Answers](#8-answers)

---

## 1. Cloud Cost Model

### Main Cost Drivers

| Category | Examples | Typical % |
|----------|---------|-----------|
| Compute | VMs, Kubernetes nodes, Lambda | 40-60% |
| Storage | Disks, S3/Blob, database storage | 15-25% |
| Database | Managed DB instances (RDS, Cloud SQL) | 15-25% |
| Network | Data transfer (egress), load balancers | 5-15% |
| Other | Monitoring, logging, DNS, secrets | 5-10% |

### Pay-Per-Use vs. Reserved

```
On-Demand: Full price, no commitment → dev/test, spiky workloads
Reserved (1-3 years): 30-70% discount → stable production workloads
Spot/Preemptible: 60-90% discount → batch jobs, fault-tolerant workloads
Savings Plans: Flexible reserved → commitment to $/hour, not specific instances
```

---

## 2. Right-Sizing

### The Problem

```
Common: Teams request "large" instances "just in case"
Reality: Most pods use 10-20% of requested resources

Wasted resources = wasted money
```

### Kubernetes Resource Optimization

```yaml
# Before (over-provisioned):
resources:
  requests: { cpu: "2000m", memory: "4Gi" }
  limits:   { cpu: "4000m", memory: "8Gi" }
# Actual usage: 200m CPU, 512Mi memory → 90% waste!

# After (right-sized based on monitoring):
resources:
  requests: { cpu: "250m", memory: "512Mi" }
  limits:   { cpu: "500m", memory: "1Gi" }
```

### Tools

| Tool | What It Does |
|------|-------------|
| Kubernetes VPA | Recommends resource requests based on actual usage |
| Goldilocks | Dashboard showing VPA recommendations per pod |
| kubecost | Full cost allocation and optimization |
| Cloud provider tools | AWS Cost Explorer, Azure Cost Management |

---

## 3. Spot/Preemptible Instances

```
Spot instances: 60-90% cheaper, but can be terminated with 2 min notice

Good for:                    Bad for:
✅ Batch processing          ❌ Databases
✅ CI/CD runners             ❌ Stateful services
✅ Stateless web servers     ❌ Single-instance services
✅ Data processing           ❌ Long-running transactions

Strategy: Mix on-demand (baseline) + spot (burst capacity)
  Node pool 1: On-demand (2 nodes, always running)
  Node pool 2: Spot (0-10 nodes, auto-scaled)
```

---

## 4. Serverless Economics

```
Traditional: Pay for servers 24/7 even at 0 traffic
Serverless:  Pay per invocation (zero cost at zero traffic)

Break-even analysis:
  Low traffic (<1M requests/month): Serverless cheaper
  High traffic (>10M requests/month): Containers cheaper
  Variable traffic with idle periods: Serverless much cheaper
```

| Use Case | Serverless (Lambda/Functions) | Containers (K8s) |
|----------|:---:|:---:|
| Low/variable traffic | ✅ | ❌ |
| Consistent high traffic | ❌ | ✅ |
| Event-driven processing | ✅ | ⚠️ |
| Long-running processes | ❌ | ✅ |
| Cold start sensitive | ❌ | ✅ |

---

## 5. FinOps Practices

### Cost Allocation

Tag everything:
```
tags:
  team: "order-team"
  service: "order-service"
  environment: "production"
  cost-center: "CC-1234"
```

### Cost Monitoring

```
Weekly cost review:
  - Top 5 most expensive services
  - Week-over-week change
  - Anomaly detection (unexpected spikes)
  - Unused resources (idle VMs, unattached disks)
```

### Architecture Cost Patterns

| Decision | Cost Impact |
|----------|------------|
| Multi-AZ deployment | 2-3x compute cost for redundancy |
| Cross-region replication | Data transfer + storage costs |
| Microservices (vs monolith) | More load balancers, more networking |
| Event streaming (Kafka) | Broker instances + storage |
| Observability (logs/traces) | Storage grows fast — set retention policies |

### Quick Wins

1. Delete unused resources (idle VMs, old snapshots, unattached disks)
2. Right-size based on actual usage data
3. Use reserved instances for stable workloads
4. Set retention policies on logs and backups
5. Use auto-scaling (don't run peak capacity 24/7)
6. Choose cheaper storage tiers for cold data

---

## 6. Exercises

### Exercise 1: Cost Analysis

You're running 8 microservices on Kubernetes with 10 nodes (4 CPU, 16GB each). Monthly bill: $15,000. Investigate and propose optimizations:
1. Analyze resource utilization
2. Identify right-sizing opportunities
3. Propose spot instance strategy
4. Calculate projected savings

### Exercise 2: Build vs. Buy

Compare build vs. buy for: managed Kafka vs. self-hosted, managed PostgreSQL vs. self-hosted on K8s. Include: infrastructure cost, operational cost (team hours), reliability, scaling complexity.

---

## 7. Self-Check Questions

1. What are the main cloud cost categories?
2. How do you right-size Kubernetes resources?
3. When should you use spot/preemptible instances?
4. When is serverless more cost-effective than containers?
5. What are the key FinOps practices?

---

## 8. Answers

### Answer 1
Compute (VMs, K8s nodes, serverless) — typically 40-60%. Storage (disks, object storage, database storage) — 15-25%. Database (managed DB instances) — 15-25%. Network (data transfer out, load balancers, DNS) — 5-15%. Other (monitoring, logging, secrets management) — 5-10%.

### Answer 2
(1) Monitor actual resource usage over 2+ weeks. (2) Compare requests/limits to actual usage. (3) Use VPA recommendations or tools like Goldilocks. (4) Set requests to p95 of actual usage. (5) Set memory limits to 1.5-2x requests. (6) Review regularly — usage patterns change.

### Answer 3
Use spot when: workloads are fault-tolerant, stateless, and can handle interruptions. Good for: batch processing, CI/CD, stateless web servers with multiple replicas. Bad for: databases, stateful services, anything that can't tolerate a 2-minute shutdown notice. Strategy: mix on-demand for baseline, spot for burst.

### Answer 4
Serverless is cheaper when: traffic is low or highly variable, there are long idle periods (nights, weekends), workloads are event-driven and short-lived. Containers are cheaper for: consistently high traffic, long-running processes, workloads sensitive to cold start latency.

### Answer 5
(1) Tag all resources (team, service, environment, cost-center). (2) Weekly cost reviews. (3) Anomaly detection on spending. (4) Right-sizing based on actual usage. (5) Reserved instances for stable workloads. (6) Automated cleanup of unused resources. (7) Set retention policies. (8) Architectural cost awareness in design decisions.

---

## Recommended Reading List

### Must Read (Priority Order)
1. **"Designing Data-Intensive Applications"** — Martin Kleppmann *(finish the second half!)*
2. **"Fundamentals of Software Architecture"** — Mark Richards & Neal Ford
3. **"Building Microservices"** (2nd Ed) — Sam Newman
4. **"Release It!"** (2nd Ed) — Michael Nygard

### Highly Recommended
5. **"Cloud Native Patterns"** — Cornelia Davis
6. **"System Design Interview Vol 1 & 2"** — Alex Xu *(you've read Vol 1)*
7. **"The Phoenix Project"** — Gene Kim
8. **"Team Topologies"** — Matthew Skelton & Manuel Pais
9. **"Software Architecture: The Hard Parts"** — Neal Ford, Mark Richards

### Deep Dives
10. **"Kubernetes in Action"** (2nd Ed) — Marko Lukša
11. **"Domain-Driven Design Distilled"** — Vaughn Vernon
12. **"Designing Distributed Systems"** — Brendan Burns
13. **"Site Reliability Engineering"** (Google SRE Book) — free online

### Online Resources
- [Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/) — excellent reference
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [12-Factor App](https://12factor.net/)
- [CNCF Landscape](https://landscape.cncf.io/)
- [Martin Fowler's Blog](https://martinfowler.com/)
- [InfoQ Architecture Podcast](https://www.infoq.com/architecture-design/)

---

*Previous: [13 - Architecture Decisions](13-architecture-decisions.md)*  
*This is the final lesson. Return to: [00 - Knowledge Map](00-architect-knowledge-map.md)*