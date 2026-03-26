# Prompt: Lesson 13 — Architecture Decision Records & Documentation

```markdown
You are an expert cloud-native software architect and technical educator. Create **Lesson 13: Architecture Decision Records & Documentation**.

### Context
Course for experienced Java developers. Deep, practical. Focus on the "soft skills" and documentation practices of architecture.

### Topic Coverage

1. **Architecture Decision Records (ADRs)**
   - Why ADRs matter (the "6 months later" problem)
   - Michael Nygard template (Status, Context, Decision, Alternatives, Consequences)
   - Complete ADR example (Kafka vs RabbitMQ for event streaming)
   - ADR lifecycle (Proposed → Accepted → Superseded/Deprecated/Rejected)
   - Storage: docs/adr/ in the repository

2. **C4 Model for Architecture Diagrams**
   - Four levels of zoom (System Context, Container, Component, Code)
   - When to use each level (stakeholders, architects, developers)
   - Level 1 example: E-commerce with external systems
   - Level 2 example: Internal services, databases, message brokers
   - Diagrams as Code (Structurizr, PlantUML, Mermaid)
   - Mermaid example that renders in GitHub

3. **Architecture Review Process**
   - When to trigger a review (7 triggers)
   - Review checklist (8 items)
   - Architecture Review Board (ARB) structure

4. **Fitness Functions**
   - Automated architectural property verification
   - Examples (6 fitness functions)
   - Integration with CI/CD

5. **Trade-Off Analysis (ATAM)**
   - Architecture Trade-Off Analysis Method
   - Quality attribute impact table example (event sourcing decision)
   - Documenting trade-offs explicitly

6. **Technical Debt Management**
   - Identifying and categorizing tech debt
   - Tech debt quadrant (Martin Fowler: Reckless/Prudent × Deliberate/Inadvertent)
   - Tracking and prioritizing

7. **Communicating Architecture**
   - Different audiences (executives, developers, ops)
   - Adapting message and detail level
   - Architecture presentations template

### Code Examples Required
1. ArchUnit fitness function tests (Java)
2. Mermaid diagram code for C4 Level 2
3. ADR template as markdown

### Diagrams Required (minimum 6)
C4 Level 1 (System Context), C4 Level 2 (Container), ADR lifecycle, Architecture review process flow, Trade-off analysis table, Fitness function in CI/CD pipeline

### Real-World Case Study
Write 3 ADRs for a microservices migration project: database choice, communication protocol, deployment strategy. Include full context, alternatives, and consequences.

### Exercises (3)
1. Write 3 ADRs (PostgreSQL vs MongoDB, Istio vs Resilience4j, REST vs gRPC)
2. Create C4 diagrams for food delivery platform (Level 1 + Level 2)
3. ATAM analysis for monolith vs microservices for startup MVP

### Self-Check Questions (4)
Cover: ADR purpose, C4 model levels, architecture review triggers, fitness functions.

### References
Include: "Documenting Software Architectures" (Clements et al.), C4 model (c4model.com), Michael Nygard ADR template, "Fundamentals of Software Architecture" (Richards & Ford), ThoughtWorks Technology Radar.