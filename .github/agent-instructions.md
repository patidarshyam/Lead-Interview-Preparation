# Agent Instructions — Interview Prep Repository

> Load this file at the start of every agent session before making any changes.

---

## Repository Purpose

This is a personal **Interview Prep Q&A repository** for a Lead Software Engineer II targeting a Senior Lead Software Engineer I promotion, and also preparing for external job interviews.

**Workspace path:** `C:\Users\YTKCKA9\OneDrive - Deere & Co\Desktop\sp\Technical\`

---

## Candidate Profile

| Field | Value |
|-------|-------|
| Current Role | Lead Software Engineer II |
| Target Role | Senior Lead Software Engineer I |
| Experience | 8.6 years total, 2.5 years in current role |
| Domain | Agriculture Product Development |
| Stack | Java 8–21, Spring Boot, AWS (ECS Fargate, SQS FIFO, VPC, S3, Redis ElastiCache), React, MySQL |

### Key Achievements (use these in behavioral answers — generalized, no company names)
1. DB migration analysis — recommended deferral, saving significant cost/time
2. VPC Peering RCA — found root cause of cross-service API failures
3. Redis cache design for burst traffic (Context API)
4. Okta + WAF implementation for internal support tool
5. Orphan data cleanup — production data analysis
6. AI agent for integration testing
7. AI agent for team onboarding
8. GitHub Copilot adoption champion
9. Hackathon organizer
10. PR reviews across team

---

## Answer Format (MANDATORY)

All new Q&A answers must follow this format:

**Definition → Mechanism → Benefits / Trade-offs**

Written as a **single flowing paragraph** (not bullet-heavy):
- **Definition:** What it is (1–2 lines, clear, no jargon overload)
- **Mechanism:** How it works internally — key rules, flow, behavior, small code example if needed
- **Benefits/Trade-offs:** Why we use it, advantages, limitations or drawbacks

**Example:**
> HashMap is a data structure that stores key-value pairs and allows fast retrieval → it uses hashing to convert a key into an index and stores entries in buckets, handling collisions using linked list or tree → it provides O(1) average performance but can degrade to O(n) in case of poor hashing and is not thread-safe.

**Rules:**
- Single flow answer, not bullet-heavy
- Crisp but complete — cover what + how + why
- Avoid unnecessary theory unless the question asks for deep dive
- Include code snippets for implementation questions (Java/SQL/React)
- Use comparison tables for vs-type questions

**Exception:** Behavioral answers use **STAR format** (Situation → Task → Action → Result).

---

## Repository Structure

```
Technical/
├── README.md                              ← Master index (sequential study path)
├── progress-tracking.md                   ← What's done / what's next
├── CONTINUATION-PROMPT.md                 ← Legacy context (superseded)
├── 01-behavioral/                         ← STAR format answers (promotion-focused)
├── 02-technical/                          ← Core technical (files 01–10 = study sequence)
│   ├── 01-core-java.md
│   ├── 02-java-spring-boot.md
│   ├── 03-databases-sql.md
│   ├── 04-react-frontend.md
│   ├── 05-aws-services.md
│   ├── 06-redis-caching.md
│   ├── 07-devops-cicd.md
│   ├── 08-system-design.md
│   ├── 09-debugging-and-rca.md
│   └── 10-security.md
├── 03-innovation/                         ← AI, tooling, innovation
├── 04-job-interview/                      ← External job interview prep
├── 05-company-interviews/                 ← Real interview Q&A by company
├── assets/images/                         ← Diagrams (see README inside)
├── .github/agent-instructions.md         ← This file
└── resources/output/
    ├── categorized_by_topics.txt          ← SOURCE FILE (280,735 lines, ~156 conversations)
    └── summary.txt
```

---

## File Conventions

| Rule | Detail |
|------|--------|
| No proprietary info | Use "the product", "the team", "the org", "the platform" |
| Q&A numbering | `## Q1:` at top level, `### Q1:` inside sections — sequential per file |
| File naming | Lowercase, hyphenated with sequence prefix: `01-core-java.md`, `02-java-spring-boot.md` (02-technical/ files follow 01–10 study order) |
| Code blocks | Use fenced code blocks with language tag: ` ```java `, ` ```sql `, ` ```jsx ` |
| Comparison tables | Use markdown tables for vs-type questions |
| README update | Always update `README.md` when adding a new file |

---

## Source File — How to Search

The source file `resources/output/categorized_by_topics.txt` contains ~156 ChatGPT conversation exports.

**Format inside source file:**
```
[From: Conversation Title]

Question NNN:
<question text>

Answer:
<answer text>

---
```

**PowerShell commands to navigate it:**
```powershell
$file = "C:\Users\YTKCKA9\OneDrive - Deere & Co\Desktop\sp\Technical\resources\output\categorized_by_topics.txt"

# Find a conversation by title
Select-String -Path $file -Pattern "^\[From: Spring Boot" | Select-Object -First 3

# Read content at a specific line range
Get-Content $file | Select-Object -Skip 140029 -First 300

# List all conversation titles
Select-String -Path $file -Pattern "^\[From:" | Select-Object -First 50
```

> **Note:** The file has duplicate conversations in places. Always check for duplicates before extracting.

---

## Adding New Q&A — Checklist

1. Read the target file to know the current last Q number
2. Append new Q&A using the correct numbering (`## Q<n+1>:`)
3. Follow the **Definition → Mechanism → Benefits/Trade-offs** answer format
4. Add code examples for implementation questions
5. Update `README.md` if a new file was created (add link + Q&A count)
6. Update `progress-tracking.md` to reflect what was added

---

## Git

The repo is initialized but has **no commits yet**. All files are untracked.

To do an initial commit:
```powershell
cd "C:\Users\YTKCKA9\OneDrive - Deere & Co\Desktop\sp\Technical"
git add .
git commit -m "Initial commit: 139 Q&A across 22 files"
```
