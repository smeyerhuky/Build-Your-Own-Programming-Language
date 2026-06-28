---
type: PRD
title: "LangCraft Platform — Product Requirements Document"
description: Full PRD for the LangCraft platform covering problem statement, target users, goals, non-goals, feature requirements, and success metrics.
tags: [prd, product, requirements, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
version: 1.0
status: Draft
owner: Clinton L. Jeffery (CTO)
---

# Product Requirements Document — LangCraft Platform v1.0

| Field | Value |
|---|---|
| Document version | 1.0 |
| Status | Draft |
| Owner | Clinton L. Jeffery |
| Last updated | 2026-06-28 |
| Review cycle | Monthly until v1.0 ships; quarterly thereafter |

---

## 1. Problem statement

### 1.1 The developer problem

Building a programming language — or even a modest domain-specific language — requires a deep, hard-to-acquire skill set: compiler theory, tool configuration, parser generators, abstract syntax trees, type systems, code generation. The resources to learn this are:

- **Books** (including ours) — comprehensive but self-directed; no feedback, no interactivity, no tooling.
- **University courses** — gated behind enrollment; not available to working professionals.
- **Open-source frameworks** (ANTLR, Racket, LLVM) — powerful but steep learning curves; not designed for learning.

The result: a large and growing population of developers who *need* to build a DSL or compiler but have no accessible path to do so.

### 1.2 The enterprise problem

Companies in cloud infrastructure, data engineering, security policy, AI workflow orchestration, and fintech are increasingly building internal DSLs. Building a language in-house today means:
- Hiring compiler engineers at $200K+/year.
- Spending 6–18 months building toolchain infrastructure.
- Getting no community, no documentation ecosystem, no package registry.

There is no **developer-tools SaaS product** for this problem.

### 1.3 The opportunity

The *Build Your Own Programming Language* book has already validated that there is a large, latent audience. The OKF repository structure has validated that LLM-assisted learning dramatically reduces setup friction. The enterprise need is real and growing. The gap is a product that connects these dots.

---

## 2. Target users

### Primary: The Self-Directed Developer Learner

**Who they are:** Software engineers with 3+ years of experience, intermediate Java or Python, curious about how programming languages work. Often motivated by needing to build a DSL for a work project or by intellectual curiosity.

**Their goal:** Build a working compiler or interpreter; understand how real compilers work.

**Their pain:** Environment setup takes hours; the gap between reading and doing is large; no feedback loop; gets stuck and has no help.

### Secondary: The University Instructor

**Who they are:** CS faculty teaching compilers, programming languages, or software engineering courses. Want to use a practical, modern curriculum.

**Their goal:** Give students a structured, runnable curriculum with exercises and auto-grading.

**Their pain:** Existing tools are old (bison/flex), examples are too simple or too complex, setup differs across student machines.

### Tertiary: The Enterprise DSL Builder

**Who they are:** Senior engineers at companies building internal DSLs for config, query, workflow, or policy. Often have strong CS fundamentals but have not built a compiler from scratch.

**Their goal:** Production-ready DSL toolchain with IDE integration, CI/CD, and documentation generation.

**Their pain:** No off-the-shelf tool meets their needs; starting from scratch wastes months; no community or ecosystem.

---

## 3. Goals

### Must have (P0)
- A new user can go from account creation to a running J0 program in **under 10 minutes**.
- The platform supports the complete J0 compiler pipeline from lexer through VM interpreter.
- The platform provides interactive exercises with automated verification for all 5 onboarding phases.
- An LLM assistant is integrated and aware of the course content.

### Should have (P1)
- Instructors can create course cohorts, assign exercises, and view student progress.
- The platform supports extending J0 (adding keywords, operators, statements) with guided scaffolding.
- A VS Code extension provides syntax highlighting, error squiggles, and hover docs for J0.
- The platform generates a shareable "portfolio link" for completed projects.

### Nice to have (P2)
- Support for x86-64 native code generation in the sandbox.
- A package registry for sharing J0 language extensions.
- Enterprise SSO (SAML/OIDC) for team accounts.
- An API for embedding LangCraft sandboxes in external documentation sites.

---

## 4. Non-goals (v1.0)

- We are **not** building a general-purpose cloud IDE (that is VS Code.dev / GitHub Codespaces — we integrate with it, not compete with it).
- We are **not** building a compiler for a production-grade general-purpose language. J0 is a teaching language.
- We are **not** building our own parser generator. We use JFlex and BYacc/Cup.
- We are **not** building mobile apps. Web-first only.
- We are **not** supporting languages other than J0 in v1.0 (multi-language DSL support is a v2 feature).

---

## 5. Feature requirements

### 5.1 Cloud sandbox (P0)

| Requirement | Details |
|---|---|
| F-001 | Authenticated users can write J0 source in a web editor |
| F-002 | The sandbox runs the full J0 pipeline (lex → parse → AST → TAC → VM) server-side |
| F-003 | Compiler output (tokens, AST, errors, stdout) is displayed in the browser |
| F-004 | Each phase can be inspected individually (show token stream, show AST, etc.) |
| F-005 | Sandbox sessions are saved and restored on login |
| F-006 | Time limit: 10 seconds per compilation; memory limit: 256 MB |

### 5.2 Interactive exercises (P0)

| Requirement | Details |
|---|---|
| F-010 | Each of the 5 onboarding phases has at least 3 guided exercises |
| F-011 | Each exercise has a specification, starter code, and an automated test harness |
| F-012 | The test harness verifies correct output (token count, AST shape, program stdout) |
| F-013 | Learners can see which tests pass/fail with line-level feedback |
| F-014 | A "hint" system provides progressive hints without giving away the answer |

### 5.3 LLM assistant (P0)

| Requirement | Details |
|---|---|
| F-020 | An LLM assistant is available in the sandbox |
| F-021 | The assistant has access to the OKF course content as its context |
| F-022 | The assistant can explain compiler errors in plain English |
| F-023 | The assistant can suggest fixes for common setup and compilation errors |
| F-024 | The assistant does not have access to other users' code |

### 5.4 VS Code extension (P1)

| Requirement | Details |
|---|---|
| F-030 | Extension provides J0 syntax highlighting |
| F-031 | Extension runs the J0 lexer and surfaces lexer errors as squiggles |
| F-032 | Extension provides hover documentation for J0 keywords and operators |
| F-033 | Extension provides a "Run in LangCraft Sandbox" command |

### 5.5 Instructor/cohort tools (P1)

| Requirement | Details |
|---|---|
| F-040 | Instructors can create a cohort and invite students by email or link |
| F-041 | Instructors can assign specific phases/exercises to a cohort |
| F-042 | Instructors can view per-student progress dashboards |
| F-043 | Instructors can extend deadlines per student |
| F-044 | The platform generates an exportable grade report (CSV) |

---

## 6. Success metrics

### Acquisition
- Time to first build (TTFB): **< 10 minutes** for 80 % of new users.
- 30-day activation rate: **> 50 %** of signups complete Phase 1.
- Monthly new signups from organic search: **+20 % MoM** in first 6 months.

### Engagement
- Phase completion rate: **> 30 %** of activated users complete all 5 phases within 30 days.
- DAU/MAU: **> 0.15** (target 0.25 by month 6).
- LLM assistant satisfaction score: **> 4.0 / 5.0** on post-session surveys.

### Revenue
- Free-to-Pro conversion: **> 5 %** within 90 days of signup.
- MRR at 6 months post-launch: **$10K**.
- First enterprise pilot: signed within **12 months** of launch.

### Retention
- Month-1 retention: **> 40 %**.
- Pro subscriber annual churn: **< 15 %**.

---

## 7. Constraints and dependencies

| Constraint | Detail |
|---|---|
| JFlex / BYacc dependency | The sandbox must run JFlex 1.9.x and BYacc-compatible parser generation. Both are open-source; no license cost. |
| JDK version | J0 compiler requires JDK 11+. Cloud sandbox must provision JDK 11 or 17. |
| Unicon runtime | The Unicon version of the J0 compiler requires Unicon 13+. Unicon must be containerised in the sandbox. |
| Packt Publishing | LangCraft may not reproduce book text verbatim. All course content is original. |
| LLM provider | Initial integration targets OpenAI GPT-4o or Anthropic Claude 3.5. Provider API costs are a significant COGS item. |

---

## 8. Open questions

| # | Question | Owner | Due |
|---|---|---|---|
| OQ-1 | Should the free tier allow unlimited compilations or impose a daily quota? | CEO (TBD) | Before pre-seed close |
| OQ-2 | Should we support Python bindings for J0 in v1.0 to reach a wider learner audience? | Clinton | Q4 2026 |
| OQ-3 | What is the right LLM context window strategy — full OKF bundle or per-page RAG? | Engineering | Q4 2026 |
| OQ-4 | Can we negotiate a content licensing deal with Packt to surface book excerpts in the platform? | CEO (TBD) | Q1 2027 |
