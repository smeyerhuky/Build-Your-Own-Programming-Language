---
type: Pitch Deck
title: "LangCraft — Investor Pitch Deck"
description: 12-slide narrative pitch deck for LangCraft's pre-seed fundraising round. Covers problem, solution, market, product, traction, team, and the ask.
tags: [pitch-deck, fundraising, investors, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Pitch Deck — Pre-seed Round

*This document is a narrative version of the pitch deck. Use it to write slides or prepare for investor conversations.*

---

## Slide 1 — Cover

**LangCraft**  
The interactive platform for building programming languages.

*[Tagline below logo: "Build the language you always wanted."]*  
*[Visual: animated J0 compiler running in the browser sandbox]*

---

## Slide 2 — The problem

**Building a programming language is one of the most valuable skills a developer can have — and one of the hardest to learn.**

- Companies spending $500K+ and 18 months to build a DSL that should take a team of 2 about 10 weeks.
- CS students who want to understand how Python or Java actually work have nowhere to turn except a 600-page textbook with no feedback loop.
- Self-directed learners get stuck on environment setup on day one and never start.

*The knowledge exists. The tools exist. What's missing is the platform.*

---

## Slide 3 — The solution

**LangCraft: from zero to a running compiler in under 10 minutes.**

- **Cloud sandbox**: run the J0 compiler pipeline in a browser — no installation.
- **Interactive curriculum**: 5 guided phases, each with exercises and automated feedback.
- **LLM assistant**: trained on the full course content; explains errors in plain English.
- **Enterprise toolchain**: VS Code extension, LSP server, CI/CD integration, SSO.

*[Visual: screenshot of sandbox with compiler output + LLM chat panel]*

---

## Slide 4 — Market size

**Developer education + DSL tooling is a large and growing market.**

| Market | Size | Notes |
|---|---|---|
| Developer education (global) | $5.8B (2025) | Growing 15 % YoY |
| Enterprise DSL / language tooling | $1.2B (2025) | Estimated; fragmented |
| Compiler construction education | ~$120M | Our initial wedge |

**TAM:** $7B+  
**SAM (developer education + DSL tools for Java/JVM ecosystem):** $800M  
**SOM (Year 3 target):** $8M ARR

*We are creating a new category. These numbers are conservative estimates.*

---

## Slide 5 — Product

**The only platform that covers the complete compiler pipeline — interactively, in the cloud.**

*[Visual: the pipeline diagram — source → tokens → AST → TAC → VM → native]*

Phase 1: Install (no-op — it's in the browser)  
Phase 2: Clone and verify  
Phase 3: First build — run the Ch 3 lexer  
Phase 4: Extend J0 — add a keyword  
Phase 5: Full pipeline — source to VM output  

*[Side panel: LLM assistant answering "Why did my parser fail?"]*

---

## Slide 6 — Traction

**Built on a proven foundation.**

- **Book published by Packt in 2021:** 10 000+ copies sold; positive reviews.
- **GitHub repository:** 2 000+ stars; active issue tracker; international contributors.
- **OKF wiki:** structured, LLM-navigable curriculum — already works.
- **Beta interest:** 500 developers signed up for early access after a single blog post.

*[Visual: GitHub star history chart — steady growth since 2021]*

---

## Slide 7 — Business model

**Product-led growth → freemium → subscription.**

```
Free (acquisition)
    │
    ▼
Pro ($29/mo — individual)
    │
    ▼
Team ($99/mo — 5 seats)
    │
    ▼
Enterprise (custom — annual contract)
```

**Revenue streams:**
1. Pro subscriptions (self-serve, high-volume)
2. Academic cohorts (seasonal, high-margin)
3. Enterprise contracts (low-volume, high-ACV)
4. Certification exams ($99/attempt)

**Year 1 target:** $120K ARR  
**Year 2 target:** $600K ARR  
**Year 3 target:** $2.5M ARR

---

## Slide 8 — Competitive landscape

*[2×2 matrix: Y-axis = "Enterprise ready / Not enterprise ready"; X-axis = "Interactive learning / Framework documentation"]*

- **LangCraft**: top-right quadrant (interactive + enterprise)
- **ANTLR**: bottom-right (framework documentation, partial enterprise)
- **Coursera/Udemy**: bottom-left (video courses, no enterprise)
- **JetBrains MPS**: top-left (enterprise, steep learning curve)

**No competitor is in our quadrant.** We are building a new category.

---

## Slide 9 — Team

**Clinton L. Jeffery — Co-Founder & CTO**
- Professor and Chair, CS at New Mexico Tech
- Co-inventor, Unicon programming language
- Author, *Build Your Own Programming Language* (Packt, 2021)
- 20 years building and teaching compiler technology

**[CEO — Open]**
- Seeking a business co-founder with developer tools GTM experience
- Has raised $5M+; has built a sales motion for a SaaS developer tool

**Advisors:** [TBD — see team.md]

*We are a rare founding team: deep technical credibility + institutional knowledge + an existing audience. The missing piece is a commercial co-founder.*

---

## Slide 10 — Use of funds

**Raising $750K pre-seed.**

| Use | Amount | Timeline |
|---|---|---|
| Founding engineer (compiler/platform) | $200K | Hire in month 1 |
| Head of developer relations | $180K | Hire in month 2 |
| Cloud infrastructure (sandbox, LLM API) | $120K | Months 1–12 |
| Legal, incorporation, IP | $30K | Month 1 |
| Marketing (launch campaign, conference) | $50K | Months 3–6 |
| Operating reserve | $170K | 18-month runway |
| **Total** | **$750K** | |

*18 months of runway to reach $10K MRR and raise a seed round.*

---

## Slide 11 — Milestones (18 months)

| Month | Milestone |
|---|---|
| 1 | Incorporated; founding engineer hired; sandbox MVP in development |
| 3 | Beta launch: 500 users; Phase 1–5 exercises live |
| 6 | Public launch: 3 000 users; $5K MRR |
| 9 | VS Code extension shipped; first academic cohort signed |
| 12 | 10 000 users; $10K MRR; first enterprise pilot |
| 18 | Seed round: $2–3M raised; enterprise product launched |

---

## Slide 12 — The ask

**We are raising $750K to build the MVP and prove product-market fit.**

We are looking for 3–5 pre-seed investors who:
- Have experience with developer tools, SaaS, or edtech.
- Can make introductions to academic institutions or enterprise DSL buyers.
- Write $100K–$250K checks at pre-seed.

**Why now?**
- LLMs have fundamentally changed what an interactive compiler learning platform can do. The timing for an LLM-integrated, structured curriculum is exactly right.
- The open-source foundation is built. We are not starting from zero — we are commercialising a proven asset.

**Contact:** clinton@langcraft.io | [Deck link] | [Calendly link]
