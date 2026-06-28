---
type: Buyer Persona
title: "LangCraft — Buyer Personas"
description: Four buyer personas for LangCraft covering self-directed learners, university instructors, enterprise DSL engineers, and indie language builders.
tags: [personas, sales, buyers, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Buyer Personas

LangCraft has four distinct buyer personas. Each has different goals, different pains, and requires a different message.

---

## Persona 1 — "The Curious Engineer"

**Name:** Marcus  
**Title:** Senior Software Engineer  
**Company:** Mid-size SaaS company (200–1000 employees)  
**Location:** Austin, TX  
**Age:** 32

### Background
Marcus has been writing Java and Python for 8 years. He's technically strong, intellectually curious, and has always wondered how programming languages actually work under the hood. He tried to read a compiler textbook once, got stuck on BNF grammars, and gave up.

Recently his team started building a YAML-based configuration DSL for their deployment pipeline and ran into the limits of what a yaml parser can do. He googled "build a programming language" and found the GitHub repo.

### Goals
- Finally understand how lexers, parsers, and ASTs work.
- Be able to talk intelligently about compiler design in architecture reviews.
- Build a small DSL for his team's config pipeline.
- Add something impressive to his GitHub portfolio.

### Pains
- Gets stuck on environment setup (JFlex won't install, BYacc gives cryptic errors).
- Reading a textbook alone is slow — he needs interactivity and feedback.
- Has limited time: 1–2 hours per week to learn this stuff.
- Doesn't know where to start when he gets stuck.

### Buying behaviour
- Discovers via Google ("build your own programming language") or Hacker News.
- Signs up free, completes Phase 1–2 in first week.
- Converts to Pro when he wants unlimited sandbox time and the VS Code extension.
- Price sensitivity: moderate. Would pay $29/mo without thinking twice if the product works.

### Message that lands
> "Go from curious to capable in one weekend. Build a real compiler — no PhD required."

---

## Persona 2 — "The CS Professor"

**Name:** Dr. Priya Nair  
**Title:** Associate Professor of Computer Science  
**Company:** State university (10 000+ students)  
**Location:** Columbus, OH  
**Age:** 44

### Background
Priya teaches a compilers course every fall to 40–60 upper-division CS majors. She has used Aho/Ullman/Sethi ("the Dragon Book") for years and keeps hitting the same problems: the examples are in ancient C, student machines all have different setups, and grading 60 hand-in compiler assignments is a nightmare.

She heard about the *Build Your Own Programming Language* book at a PL research workshop and has been experimenting with it as a textbook supplement.

### Goals
- A consistent, reproducible development environment for all her students.
- Interactive exercises she can assign with clear pass/fail criteria — not just "does it compile."
- A modern curriculum using tools her students will actually encounter (Java, VS Code, GitHub).
- Reduce her time spent debugging student setup issues.

### Buying behaviour
- Discovers via academic conference, recommendation from colleague, or Google Scholar citation.
- Requests a free instructor account to evaluate the curriculum.
- Adopts for her fall cohort if the student experience is smooth and the platform can export grades.
- Pays for student seats at the academic discount (40 % off Pro). A cohort of 40 students = $696/semester.
- Institutional purchasing may require a PO and a contract.

### Message that lands
> "LangCraft gives your students a consistent, browser-based lab environment with automated grading — so you can focus on teaching, not setup."

---

## Persona 3 — "The Enterprise DSL Builder"

**Name:** Chen Wei  
**Title:** Staff Engineer, Platform Team  
**Company:** Large fintech (5 000+ employees)  
**Location:** New York, NY  
**Age:** 38

### Background
Chen's team is building an internal policy language for compliance rules. Right now, the rules are encoded in a combination of Drools (a Java-based rules engine) and raw JSON schemas. Engineers hate it — the DSL is unreadable, IDE support is terrible, and adding a new rule type requires a 3-sprint project.

Chen's VP wants a proper, typed DSL with a VS Code plugin, CI/CD integration, and documentation generation. Chen was assigned to evaluate options and estimate the build cost.

### Goals
- A production-grade DSL toolchain that the team can build on without becoming compiler experts.
- VS Code plugin with syntax highlighting, error checking, and go-to-definition.
- CI/CD integration so the DSL compiler runs in the existing GitHub Actions pipeline.
- Enterprise security: SSO, audit logs, no source code leaving the network.

### Buying behaviour
- Discovers via technical blog post, GitHub search, or reference from another fintech engineer.
- Starts a free trial; tests the sandbox with a simplified version of their policy grammar.
- Escalates to procurement when he wants enterprise features (SSO, audit log, self-hosted).
- Deal size: $2 000–$5 000/month for a team of 10–20 engineers.
- Requires MSA, DPA, security questionnaire, and SOC 2 Type II before signing.

### Message that lands
> "Ship a production DSL in weeks, not months. LangCraft gives your team the toolchain — IDE, compiler, CI/CD integration — without hiring a language engineer."

---

## Persona 4 — "The Indie Language Builder"

**Name:** Yuki Tanaka  
**Title:** Independent Software Developer / OSS Maintainer  
**Company:** Self-employed  
**Location:** Tokyo, Japan  
**Age:** 27

### Background
Yuki is building a small programming language for creative coding — something between Processing and Lua, aimed at digital artists. She has a CS degree but has never built a compiler before. She maintains a TypeScript library with 2 000 GitHub stars and is active on Hacker News.

She found the *Build Your Own Programming Language* GitHub repo via a "Show HN" post and worked through the first three chapters over a weekend.

### Goals
- Learn enough compiler theory to build the front end of her creative coding language.
- Get from "lexer" to "working interpreter" as fast as possible.
- Share her work on GitHub and grow a community around her language.

### Buying behaviour
- Power user of the free tier — will push the daily limits.
- Converts to Pro for unlimited sandbox time and the portfolio link.
- Price sensitivity: high. Would hesitate at $29/mo; responds well to the annual plan ($249/yr = $20.75/mo).
- Strong influencer: if she's happy, she'll write about it on her blog and Hacker News. Worth treating as a VIP.

### Message that lands
> "From lexer to interpreter in a weekend. LangCraft gives indie language builders the fast path from idea to running code."

---

## Summary table

| | Marcus | Dr. Priya | Chen Wei | Yuki |
|---|---|---|---|---|
| Segment | Individual learner | Academic | Enterprise | Indie builder |
| Primary pain | Setup friction + no feedback | Student setup + grading | Build cost + no tooling | Speed to working interpreter |
| Decision maker | Self | Faculty + dept | Staff engineer + VP | Self |
| Price sensitivity | Medium | Low (institutional) | Low | High |
| Tier target | Pro | Academic cohort | Enterprise | Pro (annual) |
| Discovery channel | Google, GitHub, HN | Colleague, conference | Tech blog, GitHub | HN, GitHub |
