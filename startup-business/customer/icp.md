---
type: ICP
title: "LangCraft — Ideal Customer Profile"
description: LangCraft's Ideal Customer Profile covering firmographics, technographics, trigger events, champion profile, and disqualifiers for each segment.
tags: [icp, customer, sales, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Ideal Customer Profile

The ICP defines the customers most likely to buy, succeed with, and expand their use of LangCraft. There are three distinct ICPs — one per revenue-bearing segment.

---

## ICP 1 — Individual Pro Subscriber

### Who they are

| Attribute | Profile |
|---|---|
| Job title | Software Engineer, Senior Software Engineer, Staff Engineer, or CS student |
| Experience | 2–10 years of programming; intermediate Java, Python, or C++ |
| Location | Global; English-language developers primary |
| Industry | Any tech company; also: government tech, academia, finance, data engineering |
| Employer size | Any (this is a personal purchase, not a company purchase) |
| Education | CS degree or self-taught; doesn't matter |

### Technographics

- Uses VS Code as primary editor (or IntelliJ)
- Active GitHub user with at least one public repo
- Has used Java or a JVM language at work
- Reads Hacker News, follows r/ProgrammingLanguages, or subscribes to a developer newsletter
- Has or is currently using an LLM for coding (Copilot, ChatGPT, Claude)

### Trigger events

| Trigger | Signal |
|---|---|
| 🔥 | Team is building or evaluating a DSL for configuration, query, or workflow |
| 🔥 | Enrolled in or TAing a compilers course |
| ⚡ | Curious about how Python / Java / Rust actually works internally |
| ⚡ | Read the *Build Your Own Programming Language* book and wanted more |
| ⚡ | Built a side project that needs a custom scripting language |
| ✓ | Interested in learning new technical skills for career growth |

### What they need from LangCraft

1. No setup friction — they will not spend a day configuring their environment.
2. Interactive feedback — reading alone is not enough; they need to run code.
3. LLM help — they use LLMs constantly; they expect LangCraft's assistant to know the curriculum.
4. A portfolio artifact — a certificate or GitHub project to show employers.

### Disqualifiers

- Has no interest in programming languages, compilers, or DSLs — LangCraft is not a general-purpose learning platform.
- Expects a video course format (use Udemy as a distribution channel to capture this segment; convert them to the platform).
- Wants to build a web app or mobile app (wrong product entirely).

### Champion profile

An individual Pro subscriber **is** the champion — there is no separate evaluator or budget holder. The purchase decision is self-directed and usually happens at the moment of hitting the free-tier LLM limit or completing Phase 3.

---

## ICP 2 — University / Academic Cohort

### Who they are

| Attribute | Profile |
|---|---|
| Decision maker | CS faculty member (assistant/associate/full professor) teaching a compilers course |
| Influencer | Department chair; students who report back on experience |
| Institution type | University or college with a CS, CE, or SE program |
| Course type | Compiler construction, programming languages, systems programming, software engineering |
| Cohort size | 10–100 students per course section |
| Purchase cycle | Semester-based (fall, spring); decision made 6–8 weeks before semester start |

### Technographics

- Uses standard academic tools: GitHub Classroom, Canvas or Blackboard LMS, Zoom
- Has at least one existing lab environment (VMs, Docker) that they are tired of maintaining
- May already use the *Build Your Own Programming Language* book or Dragon Book
- Tech-literate but not always current on developer tools (some faculty are 10 years behind industry tooling)

### Trigger events

| Trigger | Signal |
|---|---|
| 🔥 | Teaching a compilers course this semester and the existing setup is painful |
| 🔥 | Students keep failing to set up their environment at semester start |
| ⚡ | Colleague mentioned LangCraft at a conference |
| ⚡ | Read the Packt book and is looking for a lab complement |
| ✓ | Department wants to modernise the curriculum |

### What they need from LangCraft

1. Consistent, zero-setup environment for all students — the #1 pain they want solved.
2. Automated grading or at least pass/fail exercise verification.
3. Student progress visibility — which students are falling behind?
4. Academic pricing they can justify to their department or put on a student fee schedule.
5. Evidence of pedagogical quality (Clinton's credentials are important here).

### Disqualifiers

- Department has no courses touching compiler construction or programming languages.
- Institution has no budget for educational software (community colleges, resource-constrained schools — target with NSF grant-funded access).
- IT policy blocks cloud-hosted environments for student work (engage on self-hosted option; flag for the enterprise tier).

### Champion profile

The faculty member is the champion, evaluator, and often the budget holder (up to $2K–$5K/semester without procurement approval). The department chair may need to approve larger cohort deals.

**How to activate:** Email the faculty member directly; offer a free instructor account to evaluate for one week; offer to run a 30-minute demo with their students.

---

## ICP 3 — Enterprise DSL Builder

### Who they are (firmographics)

| Attribute | Profile |
|---|---|
| Company stage | Series B+ or public company |
| Company size | 500–10 000 employees |
| Industry | Financial services, cloud infrastructure, AI/ML platforms, data engineering, security/policy, developer tools |
| Engineering org size | 50–500 engineers |
| DSL use case | Configuration language, query language, policy/rules engine, workflow orchestration, data transformation |
| Budget | $18K–$72K/year (our range); appears in "developer tools" or "platform engineering" budget |

### Technographics

- Uses GitHub Enterprise or GitLab
- Has a VS Code standard for engineering (or IntelliJ)
- Uses GitHub Actions, Jenkins, or CircleCI for CI/CD
- Has an existing parser (likely ANTLR or custom) that they want to improve or replace
- Engineering blog that mentions internal tooling, DSLs, or "platform engineering"

### Trigger events

| Trigger | Signal |
|---|---|
| 🔥 | Team is actively building a DSL and hitting walls (parser limitations, no IDE support, no docs toolchain) |
| 🔥 | Existing DSL maintenance is a recurring pain; team wants to rewrite it properly |
| 🔥 | New platform engineering team formed; tasked with "build a proper config language" |
| ⚡ | Engineer at the company starred the GitHub repo or signed up with company email |
| ⚡ | Company published a blog post about their internal DSL / config language |
| ✓ | Company is hiring compiler engineers (signal: job posting with "ANTLR," "parser generator," "language server") |

### What they need from LangCraft

1. VS Code plugin with LSP (diagnostics, hover, go-to-definition) — non-negotiable for professional teams.
2. CI/CD integration — the DSL compiler must run in their existing pipeline.
3. Enterprise security: SSO (Okta/Azure AD), audit logs, no source code leaving their VPC.
4. Support SLA — they cannot debug a compiler outage themselves.
5. A credible technical team they can trust to support their production language.

### Disqualifiers

- Company uses an entirely custom language runtime that can't be J0-based (e.g., Lua, WebAssembly, JVM bytecode with custom extensions) — LangCraft v1 only supports J0 bytecode target.
- Company's DSL is already in production and working well; they are not in a build/rebuild phase.
- Company requires FedRAMP or IL2/IL4 accreditation (deferred to v2).
- Company is <50 engineers (too small for enterprise tier; route to Team tier instead).

### Champion profile

**Primary champion:** Staff Engineer or Principal Engineer on the platform team. They evaluate the technical product and make the "this works" determination.

**Economic buyer:** VP Engineering or Director of Platform Engineering. They approve the budget.

**Key insight:** The champion must be convinced on *technical merit* before the economic buyer is engaged. Never start with the VP — start with the engineer who will use the product.

**How to activate:** Reach out via LinkedIn or direct email with a technical angle ("I saw you're hiring a compiler engineer — we might be able to help."). Offer a 30-minute technical demo with the founding engineer present.

---

## ICP signal scoring (for outbound prioritisation)

Score each prospect out of 10 before prioritising outreach:

| Signal | Points |
|---|---|
| Company email signup on free tier | +3 |
| GitHub repo fork | +2 |
| Job posting with "parser generator" or "language server" | +3 |
| Company engineering blog post about DSL/language | +3 |
| Multiple employees starred the repo | +2 |
| Company in: fintech, cloud infra, AI platform, security | +1 |
| Company size 200–5000 engineers | +1 |

**Prioritise outreach for scores ≥ 7.**
