---
type: Mission Vision Values
title: "LangCraft — Mission, Vision & Values"
description: LangCraft's mission statement, 10-year vision, and core values that guide every product, hiring, and business decision.
tags: [mission, vision, values, company, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Mission, Vision & Values

---

## Mission

> **Make programming language creation accessible to every developer.**

Today, building a programming language — even a simple domain-specific one — requires deep expertise that most engineers don't have and can't easily acquire. Academic resources exist but are fragmented, self-directed, and hard to apply in production. LangCraft closes that gap.

We do this through:
- An **interactive educational platform** that takes developers from zero to a working compiler.
- A **production-grade toolchain** that extends J0 into real enterprise DSLs.
- An **LLM-integrated environment** that makes compiler construction as approachable as web development.

---

## Vision (10 years)

> **LangCraft is the platform every language is built on.**

In ten years, LangCraft is to programming languages what GitHub is to source code: the place where languages are designed, built, tested, deployed, and distributed. Every serious DSL project — from database query languages to policy engines to AI workflow orchestration languages — starts on LangCraft.

Specific milestones on the path to this vision:

| Year | Milestone |
|---|---|
| 2027 | 10 000 registered developers; first 10 enterprise customers |
| 2028 | LangCraft Package Registry for distributing DSL extensions |
| 2029 | LangCraft Cloud: hosted language runtimes, no ops required |
| 2030 | 100 000 developers; LangCraft becomes the reference platform for DSL curricula at 50+ universities |
| 2031 | LangCraft Enterprise: multi-tenant, SOC 2 Type II, FedRAMP-authorized |
| 2033 | The "build on LangCraft" ecosystem — third-party plugins, grammars, runtimes, and language marketplaces |
| 2035 | IPO or strategic acquisition; LangCraft has helped create 10 000 production DSLs |

---

## Core values

### 1. Rigour without rigidity

We believe in doing things right — correct compilers, well-specified languages, tested toolchains. But rigour doesn't mean bureaucracy. We ship, we learn, we iterate. A working prototype that teaches us something real is worth more than a perfect spec that ships in two years.

*In practice:* RFCs exist to capture good decisions, not to slow down good ones. A decision documented is a decision you don't have to re-litigate.

---

### 2. Openness as a default

The book is open. The repository is open. The OKF documentation is open. This is not an accident — it is strategy. Open content builds trust, community, and distribution in ways that closed products cannot.

We will build commercial products on top of an open core. The community that forms around the open core is our moat.

*In practice:* The J0 compiler, OKF wiki, and onboarding bundle are and will remain open-source. Commercial features (cloud sandbox, enterprise SSO, advanced IDE integration) sit above the open core.

---

### 3. Accessibility over sophistication

Compiler construction has a reputation for being hard, esoteric, and only for people with Ph.D.s. We actively reject that reputation. Our product, documentation, and marketing should make a working-programmer with intermediate Java skills feel immediately capable.

*In practice:* Every feature ships with a tutorial. Every error message links to a fix. Every concept page starts from first principles.

---

### 4. Developer empathy

Our users are developers. They are sceptical of marketing, allergic to friction, and deeply loyal to tools that respect their time. We earn their trust by shipping things that actually work, being honest about limitations, and never wasting their time with feature-bloat.

*In practice:* We measure onboarding time to "first successful build" with obsessive attention. Our goal: under 10 minutes from account creation to running a J0 program.

---

### 5. Long-term thinking

Clinton spent 20 years building the knowledge base that became this company. We are not building for the next quarter — we are building the infrastructure layer for the next generation of programming tools. We make decisions with a 10-year horizon.

*In practice:* We will not sacrifice user trust or technical integrity for short-term revenue. We will invest in documentation, testing, and community even when it slows us down.

---

## Decision-making framework

When a decision is genuinely hard, apply this test in order:

1. **Mission test.** Does this make programming language creation more accessible? If no, don't do it.
2. **Values test.** Does this violate any of our five values? If yes, don't do it.
3. **Reversibility test.** Is this decision hard to reverse? If yes, slow down and document the reasoning.
4. **Regret minimisation test.** Will we regret not doing this in 10 years? If yes, find a way to do it.
