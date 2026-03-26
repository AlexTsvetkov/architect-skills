# Prompts for Course Management

## Table of Contents
- [Prompt 1: Create a New Lesson](#prompt-1-create-a-new-lesson)
- [Prompt 2: Improve an Existing Lesson](#prompt-2-improve-an-existing-lesson)

---

## Prompt 1: Create a New Lesson

Use this prompt when you need to generate a brand-new lesson from scratch. Replace `{{TOPIC}}`, `{{LESSON_NUMBER}}`, and `{{LESSON_TITLE}}` with the actual values.

````markdown
You are an expert cloud-native software architect and technical educator. Your task is to create **Lesson {{LESSON_NUMBER}}: {{LESSON_TITLE}}** for a professional-level course on cloud-native architecture.

### Context about the course

This course targets mid-to-senior software engineers preparing for architect-level roles. Lessons must be **deep, technical, and practical** — not surface-level overviews. The primary technology stack is **Java / Spring Boot**, but principles should be language-agnostic where appropriate.

The existing course structure is:
```
00 - Architect Knowledge Map (overview & learning path)
01 - Cloud-Native Fundamentals
02 - Architecture Styles
03 - Microservices Deep Dive
04 - Distributed Systems & Data
05 - Containers & Orchestration
06 - Cloud-Native Patterns
07 - API Design
08 - Observability & Reliability
09 - Security
10 - Event-Driven Architecture
11 - CI/CD & GitOps
12 - Scalability & Performance
13 - Architecture Decisions
14 - Cost Optimization
```

### Reference architecture resources

When writing the lesson, use the following Microsoft Azure Architecture Center resources as a reference for best practices, patterns, and architecture styles. Align recommendations with these where applicable:

- **Architecture Best Practices:** https://learn.microsoft.com/en-us/azure/architecture/best-practices/index-best-practices
- **Cloud Design Patterns:** https://learn.microsoft.com/en-us/azure/architecture/patterns/
- **Architecture Styles:** https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/
- **Well-Architected Framework:** https://learn.microsoft.com/en-us/azure/well-architected/
- **Microservices Architecture:** https://learn.microsoft.com/en-us/azure/architecture/microservices/
- **Data Guide:** https://learn.microsoft.com/en-us/azure/architecture/data-guide/

Where a Microsoft best-practice article directly covers a topic in the lesson (e.g., caching, retry, API design), reference it explicitly and align the lesson's guidance with it.

### Requirements for the lesson

Create the lesson on the topic: **{{TOPIC}}**

The lesson **MUST** include ALL of the following sections, in this order:

1. **Title & Metadata**
   - Lesson number, title, estimated reading time, difficulty level (Intermediate / Advanced).

2. **Learning Objectives**
   - 4–6 specific, measurable learning objectives using Bloom's taxonomy verbs (analyze, design, evaluate, implement, etc.).

3. **Prerequisites**
   - List lessons or knowledge the reader should have before starting.

4. **Conceptual Foundation** (the "why")
   - Explain the core problem this topic solves.
   - Provide historical context or evolution where relevant.
   - Include at least one **ASCII or Mermaid diagram** illustrating the key concept.

5. **Deep Technical Explanation** (the "what" and "how")
   - Cover the topic in depth: theory, patterns, algorithms, protocols, trade-offs.
   - Use **comparison tables** (e.g., approach A vs. B vs. C) where applicable.
   - Include at least **2 diagrams** (architecture diagrams, sequence diagrams, flow charts) using Mermaid or ASCII.
   - Discuss **trade-offs, failure modes, and anti-patterns** explicitly.

6. **Code Examples (Java / Spring Boot)**
   - Provide **at least 3 substantial code examples** (not toy snippets — real-world patterns).
   - Each example should have:
     - A brief description of the scenario.
     - Complete, compilable code (with imports, annotations, and configuration where needed).
     - Inline comments explaining non-obvious decisions.
   - Include at least one example showing **what NOT to do** (anti-pattern) alongside the correct approach.

7. **Real-World Architecture Case Study**
   - Present a realistic scenario (e.g., an e-commerce platform, a fintech system, a SaaS product).
   - Walk through the architecture decisions, showing how the lesson's concepts apply.
   - Include a **system-level architecture diagram**.

8. **Hands-On Exercises**
   - Provide **3–5 progressively difficult exercises**, from guided to open-ended.
   - Each exercise should have:
     - Clear requirements / acceptance criteria.
     - Hints (collapsed/spoiler-tagged if possible).
     - Expected deliverables (code, diagram, ADR, etc.).

9. **Self-Check Questions**
   - 8–12 questions covering comprehension, application, and analysis levels.
   - Mix question types: multiple-choice, short-answer, scenario-based, design challenges.
   - **Provide detailed answers** for every question in a separate "Answers" subsection at the end.

10. **Key Takeaways**
    - 5–7 concise bullet points summarizing the most important concepts.

11. **Further Reading & References**
    - At least 5 references: books, papers, official documentation, conference talks.
    - Prefer authoritative sources (Martin Fowler, Chris Richardson, CNCF, AWS/GCP/Azure whitepapers).
    - **Always include** at least one relevant link from the Microsoft Azure Architecture Center (https://learn.microsoft.com/en-us/azure/architecture/) — pick the most relevant best-practice or pattern page for the lesson's topic.

12. **Navigation**
    - Links to the previous and next lessons.
    - Link back to the Knowledge Map (Lesson 00).

### Style & formatting rules

- Use Markdown with proper heading hierarchy (`#`, `##`, `###`, `####`).
- Use fenced code blocks with language identifiers (```java, ```yaml, ```bash, etc.).
- Use Mermaid fenced blocks (```mermaid) for diagrams.
- Use tables for comparisons and structured data.
- Use blockquotes (`>`) for important notes, warnings, and tips.
- Use bold for key terms when first introduced.
- Keep paragraphs focused — one idea per paragraph.
- Aim for **3,000–6,000 words** of content (excluding code blocks).
- Ensure technical accuracy — do not invent APIs or annotations that don't exist.

### Quality bar

Before finishing, verify:
- [ ] All 12 sections above are present and substantive.
- [ ] At least 5 diagrams total (Mermaid or ASCII).
- [ ] At least 3 Java/Spring Boot code examples with full context.
- [ ] At least 1 anti-pattern example with explanation.
- [ ] Trade-offs and failure modes are discussed explicitly.
- [ ] Self-check questions have detailed answers.
- [ ] Exercises are progressive in difficulty.
- [ ] All references are real and authoritative.

Now generate the complete lesson.
````

---

## Prompt 2: Improve an Existing Lesson

Use this prompt when you want to enhance, deepen, or fix an existing lesson. Replace `{{LESSON_FILE}}` with the filename (e.g., `06-cloud-native-patterns.md`) and paste the current content, or instruct the AI to read it.

````markdown
You are an expert cloud-native software architect and technical educator performing a **deep review and improvement** of an existing lesson.

### Context about the course

This course targets mid-to-senior software engineers preparing for architect-level roles. Lessons must be **deep, technical, and practical** — not surface-level overviews. The primary technology stack is **Java / Spring Boot**, but principles should be language-agnostic where appropriate.

### Reference architecture resources

Use the following Microsoft Azure Architecture Center resources as a reference when evaluating and improving the lesson:

- **Architecture Best Practices:** https://learn.microsoft.com/en-us/azure/architecture/best-practices/index-best-practices
- **Cloud Design Patterns:** https://learn.microsoft.com/en-us/azure/architecture/patterns/
- **Architecture Styles:** https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/
- **Well-Architected Framework:** https://learn.microsoft.com/en-us/azure/well-architected/
- **Microservices Architecture:** https://learn.microsoft.com/en-us/azure/architecture/microservices/
- **Data Guide:** https://learn.microsoft.com/en-us/azure/architecture/data-guide/

Where a Microsoft best-practice article directly covers a topic in the lesson, ensure it is referenced in "Further Reading" and that the lesson's guidance is aligned with it.

### The lesson to improve

File: `{{LESSON_FILE}}`

<existing_lesson>
{{Paste the full lesson content here, or instruct the AI to read the file}}
</existing_lesson>

### Improvement checklist

Analyze the lesson against **every item** below. For each item, either confirm it meets the bar or improve it:

#### A. Structure & Completeness
- [ ] **Learning Objectives** — Are there 4–6 specific, measurable objectives using Bloom's taxonomy verbs? If missing or vague, rewrite them.
- [ ] **Prerequisites** — Are prerequisites listed? Add them if missing.
- [ ] **Conceptual Foundation** — Is the "why" explained with historical context and at least one diagram? Deepen if shallow.
- [ ] **Key Takeaways** — Are there 5–7 concise summary bullets? Add or sharpen them.
- [ ] **Navigation** — Are prev/next lesson links and a link to Lesson 00 present? Add if missing.
- [ ] **Further Reading** — Are there at least 5 real, authoritative references? Add more if needed. Ensure at least one link from the Microsoft Azure Architecture Center is included.

#### B. Technical Depth
- [ ] **Trade-offs** — Are trade-offs for each approach/pattern discussed explicitly (not just "pros and cons" but nuanced analysis)? Add a comparison table if missing.
- [ ] **Failure Modes** — Are failure scenarios, edge cases, and what can go wrong discussed? Add a dedicated subsection if missing.
- [ ] **Anti-patterns** — Is at least one anti-pattern shown with code and explanation of why it's wrong? Add if missing.
- [ ] **Distributed Systems Considerations** — Where relevant, are CAP theorem implications, network partitions, consistency models, idempotency, etc. discussed?
- [ ] **Production Readiness** — Are operational concerns (monitoring, alerting, runbooks, graceful degradation) covered?

#### C. Diagrams
- [ ] Are there at least **5 diagrams** (Mermaid or ASCII)?
- [ ] Do diagrams include: at least one **architecture/component diagram**, one **sequence diagram**, and one **flow chart or decision tree**?
- [ ] Are diagrams properly labeled and referenced in the text?
- [ ] Add any missing diagram types. Prefer Mermaid syntax.

#### D. Code Examples (Java / Spring Boot)
- [ ] Are there at least **3 substantial code examples** (not toy snippets)?
- [ ] Does each example include full context (imports, class definitions, annotations, configuration)?
- [ ] Are there inline comments explaining non-obvious decisions?
- [ ] Is there at least one **anti-pattern code example** with an explanation?
- [ ] Do examples reflect **real production patterns** (error handling, logging, configuration, etc.)?
- [ ] Add missing examples. Fix any examples that use non-existent APIs or incorrect annotations.

#### E. Case Study
- [ ] Is there a **real-world architecture case study** with a scenario walkthrough and system diagram?
- [ ] If missing, add one based on a realistic domain (e-commerce, fintech, SaaS, IoT, etc.).

#### F. Exercises
- [ ] Are there **3–5 progressively difficult exercises**?
- [ ] Do exercises have clear requirements, hints, and expected deliverables?
- [ ] Is there at least one exercise that requires producing an architecture diagram or ADR?
- [ ] Add or improve exercises if they are too shallow or missing.

#### G. Self-Check Questions & Answers
- [ ] Are there **8–12 self-check questions** covering comprehension, application, and analysis?
- [ ] Is there a mix of question types (multiple-choice, short-answer, scenario-based, design challenges)?
- [ ] Are **detailed answers provided** for every question?
- [ ] Add questions and answers if missing or insufficient.

#### H. Style & Formatting
- [ ] Proper Markdown heading hierarchy (`#` > `##` > `###` > `####`).
- [ ] Code blocks have language identifiers (```java, ```yaml, etc.).
- [ ] Mermaid diagrams use ```mermaid fenced blocks.
- [ ] Tables are used for comparisons.
- [ ] Blockquotes for tips/warnings/notes.
- [ ] Bold for key terms on first use.
- [ ] Content is at least **3,000–6,000 words** (excluding code). If too short, expand.

### Output instructions

1. **First**, provide a brief **Improvement Summary** listing what was missing, weak, or incorrect, and what you changed.
2. **Then**, output the **complete improved lesson** in full — do not output diffs or partial content. The output should be the final, ready-to-use lesson file.

### Quality bar

Before finishing, verify:
- [ ] All sections from the structure checklist are present and substantive.
- [ ] At least 5 diagrams total.
- [ ] At least 3 Java/Spring Boot code examples with full context.
- [ ] At least 1 anti-pattern example.
- [ ] Trade-offs and failure modes discussed.
- [ ] Self-check questions have detailed answers.
- [ ] Exercises are progressive in difficulty.
- [ ] All references are real and authoritative.
- [ ] No invented APIs, annotations, or incorrect technical claims.

Now analyze the lesson and produce the improved version.
````

---

## Usage Tips

1. **Creating a new lesson**: Copy Prompt 1, fill in `{{TOPIC}}`, `{{LESSON_NUMBER}}`, and `{{LESSON_TITLE}}`, and submit to the AI.
2. **Improving an existing lesson**: Copy Prompt 2, fill in `{{LESSON_FILE}}`, paste the lesson content (or instruct the AI to read the file), and submit.
3. **Iterative improvement**: After the AI generates an improved lesson, you can re-run Prompt 2 on the output for a second pass — focus areas can be narrowed by editing the checklist.
4. **Batch processing**: To improve all lessons, run Prompt 2 for each lesson file sequentially (e.g., `01-cloud-native-fundamentals.md`, then `02-architecture-styles.md`, etc.).
5. **Custom focus**: If you only need to improve a specific aspect (e.g., "add more diagrams" or "add exercises"), you can edit the checklist in Prompt 2 to keep only the relevant sections.