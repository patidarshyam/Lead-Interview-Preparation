# Interview Prep — Full-Stack Java Lead (Spring Boot + AWS + React)

Structured Q&A for **promotion interviews** and **job interviews** — 139 Q&A across 22 files.

> **Answer Format:** Definition → Mechanism → Benefits / Trade-offs (single flowing paragraph)
> **Behavioral Format:** STAR — Situation → Task → Action → Result

---

## Sequential Study Path

Work through these steps in order. Each step = one file. Practice answers out loud.

---

### Phase 1 — Java Foundation

| Step | Topic | File | Q&A |
|------|-------|------|-----|
| 1 | Core Java (8–21) — Streams, Virtual Threads, Records, GC | [01-core-java.md](02-technical/01-core-java.md) | 35 |
| 2 | Java & Spring Boot — REST API, Transactions, Exception Handling | [02-java-spring-boot.md](02-technical/02-java-spring-boot.md) | 25 |
| 3 | Databases & SQL — Indexing, ACID, Connection Pooling, DDL/DML | [03-databases-sql.md](02-technical/03-databases-sql.md) | 12 |
| 4 | React & Frontend — Hooks, State, Auth Flow, Router | [04-react-frontend.md](02-technical/04-react-frontend.md) | 8 |

---

### Phase 2 — Cloud & Infrastructure

| Step | Topic | File | Q&A |
|------|-------|------|-----|
| 5 | AWS Services — ECS Fargate, SQS FIFO, VPC Peering, S3 | [05-aws-services.md](02-technical/05-aws-services.md) | 5 |
| 6 | Redis & Caching — Cluster Mode, Cache Stampede, Key Design | [06-redis-caching.md](02-technical/06-redis-caching.md) | 4 |
| 7 | DevOps & CI/CD — Docker, Deployment Strategies, IaC, Observability | [07-devops-cicd.md](02-technical/07-devops-cicd.md) | 5 |

---

### Phase 3 — Architecture & Reliability

| Step | Topic | File | Q&A |
|------|-------|------|-----|
| 8 | System Design — Export Pipelines, Caching Layers, Microservices | [system-design.md](06-system-design/system-design.md) | 7 |
| 8a | Real-World Architectures — Netflix, WhatsApp deep dive | [14-real-world-architectures.md](06-system-design/14-real-world-architectures.md) | 3 |
| 9 | Debugging & RCA — Production Debugging, Cross-Service RCA | [09-debugging-and-rca.md](02-technical/09-debugging-and-rca.md) | 4 |
| 10 | Security — OWASP Top 10, OAuth/Okta, WAF, Secrets Management | [10-security.md](02-technical/10-security.md) | 4 |

---

### Phase 4 — Innovation & Leadership

| Step | Topic | File | Q&A |
|------|-------|------|-----|
| 11 | AI & Tooling — Copilot, AI Agents, Hackathons | [ai-and-tooling.md](03-innovation/ai-and-tooling.md) | 5 |
| 12 | Ownership & Accountability — Cost-saving decisions, Going above and beyond | [ownership-and-accountability.md](01-behavioral/ownership-and-accountability.md) | 4 |
| 13 | Leadership & Influence — Design leadership, Cross-team influence | [leadership-and-influence.md](01-behavioral/leadership-and-influence.md) | 4 |
| 14 | Mentoring & Growth — Developing engineers, Onboarding, Learning culture | [mentoring-and-growth.md](01-behavioral/mentoring-and-growth.md) | 4 |
| 15 | Conflict & Challenges — Disagreements, Technical blockers, Pressure | [conflict-and-challenges.md](01-behavioral/conflict-and-challenges.md) | 4 |

---

### Phase 5 — Interview Practice

| Step | Topic | File | Q&A |
|------|-------|------|-----|
| 16 | Behavioral Common — "Tell me about yourself", Career goals | [behavioral-common.md](04-job-interview/behavioral-common.md) | 4 |
| 17 | System Design HLD/LLD — URL Shortener, Rate Limiter, Notification | [system-design-hld-lld.md](04-job-interview/system-design-hld-lld.md) | 3 |
| 18 | Coding & Problem Solving — Patterns, Complexity, Approach Framework | [coding-problem-solving.md](04-job-interview/coding-problem-solving.md) | 3 |

---

### Phase 6 — Company-Specific Drills

| Step | Company | Topics | File | Q&A |
|------|---------|--------|------|-----|
| 19 | Deutsche Bank | Multithreading, Streams, SQL, Kafka, Logic Puzzles | [deutsche-bank-interview-preparation.md](05-company-wise-interview-preparation/deutsche-bank-interview-preparation.md) | 20 |
| 20 | EPAM Systems | ConcurrentHashMap, CompletableFuture, Bean Lifecycle, React | [epam-interview-preparation.md](05-company-wise-interview-preparation/epam-interview-preparation.md) | 11 |
| 21 | Infobeans | SOAP vs REST, Hibernate, Transactions, SQL Joins, Angular | [infobeans-interview-preparation.md](05-company-wise-interview-preparation/infobeans-interview-preparation.md) | 15 |
| 22 | AgileEngine | Codility-style Streams coding test | [agileengine-interview-preparation.md](05-company-wise-interview-preparation/agileengine-interview-preparation.md) | 1 |

---

### Phase 7 — System Design (Quick Access)

> All system design content in one place — use this phase for focused system design interview prep.

| Step | Topic | File | Q&A |
|------|-------|------|-----|
| SD-1 | Microservices Concepts — API Gateway, Circuit Breaker, Saga, CQRS, Service Mesh | [13-microservices-concepts.md](02-technical/13-microservices-concepts.md) | 10 |
| SD-2 | System Design — Export Pipelines, Caching Layers, Microservices Patterns | [system-design.md](06-system-design/system-design.md) | 7 |
| SD-3 | Real-World Architectures — Netflix, WhatsApp deep dive | [14-real-world-architectures.md](06-system-design/14-real-world-architectures.md) | 3 |
| SD-4 | System Design HLD/LLD — URL Shortener, Rate Limiter, Notification | [system-design-hld-lld.md](04-job-interview/system-design-hld-lld.md) | 3 |

---

## Repository Structure

```
├── 01-behavioral/              ← STAR format behavioral answers
├── 02-technical/               ← Core technical (01–07, 09–13 in study sequence)
├── 03-innovation/              ← AI, tooling, innovation leadership
├── 04-job-interview/           ← External job interview practice
├── 05-company-wise-interview-preparation/ ← Real Q&A organized by company
├── 06-system-design/           ← System design & real-world architectures
├── assets/images/              ← Diagrams and architecture references
├── progress-tracking.md        ← What's done / what's next
└── .github/agent-instructions.md ← Agent session context
```

---

## Conventions

- **Behavioral answers** use STAR format
- **Technical answers** use Definition → Mechanism → Benefits/Trade-offs
- Add new Q&A as you practice — keep iterating
