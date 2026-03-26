# Prompt: Lesson 14 — Cost Optimization & FinOps

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 14: Cost Optimization & FinOps**.

### Context
Course for experienced Java developers. Practical focus on cloud cost management. Reference: Microsoft Well-Architected Framework - Cost Optimization, AWS Well-Architected - Cost Optimization.

### Topic Coverage

1. **Cloud Cost Model**
   - Main cost drivers table (Compute 40-60%, Storage 15-25%, Database 15-25%, Network 5-15%, Other 5-10%)
   - Pay-per-use vs Reserved vs Spot/Preemptible vs Savings Plans

2. **Right-Sizing**
   - The over-provisioning problem (typical waste: 80-90%)
   - Kubernetes resource optimization (before/after YAML)
   - Tools table (VPA, Goldilocks, kubecost, cloud provider tools)
   - Right-sizing process (monitor → analyze → adjust → repeat)

3. **Spot/Preemptible Instances**
   - Cost savings (60-90%)
   - Good vs bad use cases (table)
   - Mixed strategy: on-demand baseline + spot burst
   - Kubernetes node pool strategy

4. **Serverless Economics**
   - Break-even analysis (when serverless < containers < VMs)
   - Cost comparison table by use case
   - Cold start considerations

5. **FinOps Framework and Practices**
   - Cost allocation: tagging everything (YAML example)
   - Weekly cost review process
   - Anomaly detection
   - Architecture cost patterns table (6 decisions and their cost impact)
   - Quick wins list (6 items)

6. **Multi-Cloud and Cloud-Agnostic Considerations**
   - Vendor lock-in spectrum
   - Portable vs cloud-native services
   - When to be cloud-agnostic vs cloud-native

7. **TCO Analysis**
   - Cloud vs on-premises comparison
   - Hidden costs (operational, people, opportunity)
   - Migration cost modeling

### Code Examples Required
1. Kubernetes resource YAML (over-provisioned vs right-sized)
2. Cost allocation tags YAML
3. Spot instance node pool configuration (Kubernetes)
4. VPA recommendation configuration

### Diagrams Required (minimum 4)
Cost breakdown pie chart, Serverless vs containers break-even, Right-sizing process flow, Mixed spot + on-demand architecture

### Real-World Case Study
Optimize costs for 8-service e-commerce platform on Kubernetes: analyze $15K/month bill, identify waste, propose right-sizing, spot strategy, reserved instances. Target 40% reduction.

### Exercises (2)
1. Cost analysis and optimization proposal (8 services, 10 nodes, $15K/month)
2. Build vs buy analysis (managed Kafka vs self-hosted, managed PostgreSQL vs self-hosted)

### Self-Check Questions (5)
Cover: cost categories, right-sizing K8s resources, spot instances, serverless economics, FinOps practices.

### Recommended Reading List (include in this lesson as it's the final one)
Must-read books (4), highly recommended (5), deep dives (4), online resources (6 URLs).

### References
Include: Microsoft Well-Architected Framework (Cost), AWS Cost Optimization, FinOps Foundation, kubecost documentation, "Cloud FinOps" (J.R. Storment & Mike Fuller).