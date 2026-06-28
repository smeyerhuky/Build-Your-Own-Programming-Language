---
type: Product Roadmap
title: "LangCraft Platform — Product Roadmap"
description: Now/Next/Later product roadmap for the LangCraft platform with themes, milestones, and success criteria.
tags: [roadmap, product, milestones, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Product Roadmap

*Horizon: Q3 2026 – Q4 2027. Updated quarterly.*

---

## Themes

| Theme | Description |
|---|---|
| **Zero to build** | Eliminate all friction on the path from first visit to first running compiler |
| **Learn by doing** | Every concept is paired with a runnable exercise and automated feedback |
| **AI-augmented** | The LLM assistant knows the curriculum and reduces the cost of being stuck |
| **Enterprise-ready** | Toolchain quality, security, and integrations that satisfy professional teams |
| **Community flywheel** | Features that make sharing, collaborating, and showcasing work easy |

---

## Now (Q3–Q4 2026) — Pre-seed to MVP

**Goal:** Prove that a developer can go from zero to a working J0 compiler in under 10 minutes on LangCraft.

### Infrastructure
- [ ] Cloud sandbox: containerised J0 compiler (JDK 17 + JFlex + BYacc + Unicon) running in browser
- [ ] Web editor with J0 syntax highlighting (CodeMirror or Monaco)
- [ ] Auth: email/password + GitHub OAuth
- [ ] Per-session file storage (source files, build artifacts)
- [ ] Compilation output viewer: tokens, AST, errors, stdout

### Education
- [ ] Phase 1–5 interactive exercises ported from OKF onboarding bundle
- [ ] Automated test harness for exercises (expected output comparison)
- [ ] Hint system (3 progressive hints per exercise)
- [ ] Progress tracking per user per phase

### LLM assistant
- [ ] In-sandbox assistant backed by OpenAI GPT-4o (or Claude 3.5 Sonnet)
- [ ] System prompt seeded with OKF course content (RAG over `/wiki/` and `/startup/`)
- [ ] Error explanation mode: paste compiler error → get plain-English explanation + fix suggestion

### Pricing/billing
- [ ] Free tier (5 LLM queries/day, 60 min sandbox/day)
- [ ] Pro tier ($29/mo): Stripe integration, unlimited sandbox, unlimited LLM
- [ ] Academic verification flow (`.edu` email auto-discount)

**Success criteria:** 3 000 beta signups; TTFB < 10 min for 80 % of users; NPS ≥ 50.

---

## Next (Q1–Q2 2027) — Grow and expand

**Goal:** Build the features that turn a learner into an advocate and a pilot into a paid enterprise contract.

### Education
- [ ] Phase 6–10 exercises (extending J0 with new features; full pipeline exercises)
- [ ] Certificate of completion for each phase and for the full course
- [ ] Portfolio page: public-facing gallery of completed projects
- [ ] Instructor cohort tools: create cohort, assign exercises, view dashboards, export grades

### Tooling
- [ ] VS Code extension v1: J0 syntax highlighting + run-in-sandbox command
- [ ] VS Code extension v2: LSP integration (diagnostics, hover, go-to-definition)
- [ ] CLI tool: `langcraft compile <file.j0>` (locally installed, uses same pipeline as sandbox)

### Enterprise
- [ ] Team accounts: organisation billing, member management
- [ ] Enterprise SSO: SAML 2.0 / OIDC
- [ ] Audit log (who ran what, when)
- [ ] Private sandbox (no data leaves customer's VPC — self-hosted option)
- [ ] SLA: 99.9 % uptime commitment with credits

### Community
- [ ] LangCraft Showcase: searchable gallery of student and enterprise DSL projects
- [ ] Discussion forum integrated into the platform (phase-specific threads)
- [ ] Monthly "Language of the Month" spotlight: community-built J0 extensions

**Success criteria:** $10K MRR; 1 enterprise pilot signed; 30 % of activated users complete full course.

---

## Later (Q3–Q4 2027) — Scale

**Goal:** Cross $100K ARR; establish LangCraft as the reference platform for compiler education.

### Platform
- [ ] Language Package Registry: publish and install J0 language extensions
- [ ] Multi-language support: allow learners to build DSLs beyond J0 (bring your own grammar)
- [ ] LangCraft API: embed sandboxes in external docs, tutorials, and blog posts
- [ ] Mobile-responsive design (current v1 is desktop-only)

### Enterprise
- [ ] FedRAMP authorization (targeted at US government and defence contractors)
- [ ] Data residency options (EU, APAC)
- [ ] Enterprise analytics: language usage metrics, build frequency, error rates

### Education
- [ ] University partnership program: LangCraft as the official lab environment for CS curricula
- [ ] Localization: Spanish, Mandarin, Japanese translations of course content
- [ ] Advanced certification: LangCraft Certified Language Engineer (exam-based)

**Success criteria:** $100K ARR; 10 enterprise customers; partnerships with 5 universities.

---

## Dropped / de-prioritised

| Feature | Why dropped |
|---|---|
| Mobile app | Not worth the investment at this stage; web is responsive enough |
| General-purpose cloud IDE | Competes with VS Code.dev and GitHub Codespaces; out of scope |
| Support for LLVM backend in v1 | Too complex for the target learner; deferred to Later |
| In-platform forum (own software) | Using Discourse hosted instance instead; build vs. buy |
