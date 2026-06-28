---
type: Business Plan Section
title: "Conductor Labs — Team and Hiring Plan"
description: Founder strength-to-product mapping, honest gap analysis with mitigations, three-hire sequence (DevRel → Infra Eng → Sales Eng), advisory board recruitment plan, and key-person risk mitigations for a sole-founder Conductor Labs.
tags: [team, hiring, founder, advisor, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Team and Hiring Plan — Conductor Labs

## Founder Profile

**[Founder]** — Sole founder, CEO, and CTO.

30 years in systems architecture, developer experience, infrastructure, testing, and design. Built compilers, designed developer tools, shipped production systems at scale. IS the target customer: every feature in Conductor exists because I needed it personally and couldn't find it elsewhere. The compiler pipeline maps directly to a compiler construction course (Jeffery, Packt) — this is not a "build an AI wrapper" product; it is a PL theory product that requires 30 years of systems background to build credibly.

### Strength-to-Product Mapping

| Domain | Experience | How it maps to Conductor |
|--------|-----------|-------------------------|
| Compiler construction | 30 years systems work; currently working through Jeffery (Packt) | Built the lexer, parser, AST, type checker, IR emitter — the entire Ch3–Ch8 pipeline |
| Developer experience | Designed developer tools; knows what makes tools feel good | Web IDE, error message quality, compiler diagnostic UX |
| Systems architecture | Multi-tenant systems, distributed state, fault tolerance | Multi-tenant Postgres RLS, ECS Fargate runtime, checkpoint-resume |
| Infrastructure | Cloud infrastructure at scale | AWS ECS, RDS, ElastiCache, S3, GPU fleet (vLLM + spot interruption) |
| Testing | Built test infrastructure for large codebases | E2E test suite (TASK-115), compiler correctness test harness |
| Startups | Experience building and shipping products | Product instincts; can prioritize ruthlessly |

### Gap Analysis

The founder does not cover every function a company needs. Honest assessment:

| Gap | Severity | Mitigation |
|-----|---------|------------|
| **Sales** | Medium | Sole-founder sales through $1M ARR (Collison installation method). No SDR needed until after $1M ARR. |
| **DevRel / community** | High | Hire #1. DevRel cannot be faked at scale. Founder handles content until $1M ARR; then hire. |
| **Design / UX** | Low | Monaco Editor does the heavy lifting. UI is minimal. Founder-designed is good enough for dev tools at this stage. |
| **Finance / legal** | Low | Outsource: Stripe Atlas entity, YC-recommended attorney for IP/agreements, Pilot for bookkeeping. |
| **Enterprise sales** | Medium | Not needed until Year 2. Hire #3 (Sales Engineer) post-Series A. |
| **GPU / ML Ops** | Medium | Hire #2 (Infra Engineer) at ~$500k ARR. vLLM is relatively simple operationally; founder covers until then. |

**Key insight**: 74% of YC dev tool companies had purely technical founding teams (YC 2024 cohort data). The gaps above are all functions that can be outsourced, hired, or deferred until revenue proves the model. The core product — a correct, usable compiler — requires the founder's specific background and cannot be outsourced.

---

## Hiring Sequence

### Hire #1: Developer Relations Engineer (~Month 13, ~$1M ARR)

**Why this role first?**

At $1M ARR, the product is proven. The bottleneck shifts from "does this work?" to "do enough developers know about it?" DevRel is the highest-leverage hire for a PLG dev tools company at this stage. n8n's first hire was a community manager. Tailwind CSS hired a DevRel lead before any enterprise sales. The pattern is consistent.

**What they do:**
- Conference talks (Compiler Dev, AI Engineer, PyCon, KubeCon)
- YouTube tutorial content (3–4 videos/mo)
- Discord community management (daily presence)
- Blog posts, case studies, integration guides
- Represent Conductor at hackathons and workshops

**Profile:**
- Has shipped a developer tool and written about it publicly
- Active GitHub contributor or maintainer of something people use
- Can write code (not just talk about it) — demos need to work
- Has given conference talks or has obvious potential to
- Comfortable being the public face of the company

**Compensation (estimates for US remote):**
- $130,000–$160,000 base + 0.5–1.0% equity (4-year vest, 1-year cliff)
- Generous conference budget ($15,000/yr)
- Content creation tooling budget ($3,000/yr)

**Sourcing strategy:**
- Post in: MLOps Community, AI Engineer World's Fair Slack, PL Twitter/X
- DM DevRel engineers at Stripe, Vercel, Supabase — companies with world-class DevRel culture
- Look for: people who are already creating content about AI/agent tooling without being paid to

**Interview signal to prioritize**: Give them a 20-minute live coding task in Conductor and ask them to explain it to a camera. Can they make the compiler feel approachable?

---

### Hire #2: Infrastructure / ML Ops Engineer (~Month 18–24, ~$500k ARR)

**Why this role second?**

The GPU serving stack (vLLM + MTP speculative decoding + spot interruption handling) is operationally stable but requires constant tuning as model versions change (Gemma4 → Gemma5), as AWS spot pricing shifts, and as traffic patterns evolve. Founder can manage this at 100 MAU; it becomes untenable at 1,000 MAU with SLA commitments.

**What they do:**
- Own the vLLM cluster (Terraform, model updates, MTP config)
- Build multi-region GPU spot fleet automation
- Reliability: SLA monitoring, alerting, runbooks
- Cost optimization: spot vs. on-demand mix, reserved instance planning
- New model evaluation: benchmark new model releases against Conductor workloads

**Profile:**
- Has run vLLM or equivalent (TGI, TRT-LLM) in production
- Familiar with AWS ECS, spot interruption patterns, auto-scaling
- Can write Python infrastructure code (not just click buttons)
- Terraform experience

**Compensation:**
- $160,000–$200,000 base + 0.3–0.6% equity
- Home lab / GPU budget ($5,000/yr for personal GPU experiments)

**Sourcing:**
- Post in: MLOps Community, Hugging Face Discord, Modal Slack
- Look for: engineers who have posted "I deployed Gemma/Mistral/Llama on spot instances and here's what I learned"

---

### Hire #3: Sales Engineer (~Month 24–30, post-Series A)

**Why this role third?**

Enterprise deals require a technical person to run pilots, answer architecture questions, write integration guides, and handle procurement. A Sales Engineer lets the founder stay technical while enterprise revenue scales. This hire happens after a Series A because Enterprise deals below $50k/yr TCV don't justify a dedicated SE.

**What they do:**
- Own enterprise pilot programs (onboarding, 30-day success criteria, expansion)
- Technical discovery calls with enterprise security/architecture teams
- Write integration guides (Conductor + customer's CI/CD, VPC, SSO)
- Become the primary voice for enterprise product feedback

**Profile:**
- Has been a Solutions Architect or Sales Engineer at a developer tools company
- Can read and write code (demos, integration scripts)
- Experience with SOC2 conversations, procurement processes, security questionnaires
- Former platform engineer who moved into pre-sales

**Compensation:**
- $140,000–$170,000 base + variable ($40,000–$70,000 at quota) + 0.2–0.4% equity

---

## Advisory Board

Three targeted advisors to fill specific gaps.

### Advisor 1: Go-To-Market / Developer PLG

**Profile sought**: Founder or early GTM hire at a developer PLG company that crossed $10M ARR. Ideally: open-source-first, no-outbound, HN-driven distribution playbook.

**Target names** (example, not commitments):
- A founding engineer at n8n, Supabase, Vercel, or similar
- A founder who has done a successful Show HN launch

**What they bring**: Playbook knowledge (what HN post title works, how to structure Discord, when to hire DevRel), warm intros to investors who fund dev tools, credibility with YC and similar accelerators.

**Compensation**: 0.1–0.25% equity (4-year vest, no cliff), advisory agreement, monthly 1-hour call.

### Advisor 2: Compiler / PL Theory

**Profile sought**: Academic or industry expert in programming language theory, type systems, or compiler construction. Ideally someone who has shipped a production DSL or language.

**What they bring**: Validation of the type system design, feedback on the formal semantics in the language spec, potential paper co-authorship (academic credibility), referrals from the PL community (where 1 credible endorsement is worth 10,000 cold outreach emails).

**Compensation**: 0.05–0.10% equity, co-authorship on any academic work, invited speaker at launch events.

### Advisor 3: Enterprise SaaS / AWS Ecosystem

**Profile sought**: Someone who has navigated AWS ISV Accelerate, AWS Marketplace, and enterprise procurement cycles. Ideally: built a B2B SaaS that scaled through AWS co-sell.

**What they bring**: AWS Marketplace listing process (often 3–6 months of paperwork), warm intros to AWS field SAs, guidance on enterprise pricing and procurement structure, FedRAMP pathway if relevant.

**Compensation**: 0.1–0.15% equity, introductions to 5 AWS SAs, monthly call.

---

## Key Person Risk Mitigations

Sole founders are a key person risk. Mitigations:

| Risk | Mitigation |
|------|------------|
| Founder illness / incapacitation | All source code in GitHub (MIT-ish license for open-core components). All infrastructure as Terraform. Operations runbooks in `ops/runbooks/`. Any investor gets a dead man's switch clause (access to codebase if founder is incapacitated >90 days). |
| Burnout | $10k/mo salary from raise (replaces freelance income; no financial pressure). Quarterly "zero week" (no Slack, no issues, no calls). Advisor board provides external perspective and accountability. |
| Technical bus factor | Architecture documented in `startup/06-design/`. Compiler is well-typed Python with 80%+ test coverage. The IRWalker loop is <300 lines. A strong engineer can onboard in 2 weeks. |
| Founder-market fit loss | Monthly user interviews (15 per month). NPS tracked weekly. If NPS drops below 40 for 2 consecutive months, conduct 10 churn interviews. Product roadmap is thesis-driven, not request-driven. |

---

## Culture Principles (Pre-Hire)

These principles exist before any hire because culture is set by the founder's behavior, not by culture docs written after hiring.

1. **Ship weekly.** Every Friday, something is in production that wasn't on Thursday. Small and real beats big and speculative.

2. **Compiler first.** When product decisions conflict, the choice that makes the compiler better wins. The compiler is the product. The IDE is a demo for the compiler.

3. **Write it down.** Every significant technical decision is an RFC. Every architecture change is a design doc. Documentation is not overhead — it is how a sole founder multiplies their decisions over time and sets expectations for future hires.

4. **Users are coworkers, not metrics.** The first 100 users get a personal reply from the founder within 24 hours, every time. After 1,000 users, this becomes unsustainable — but by then, community infrastructure (Discord, GitHub Discussions) handles it.

5. **No bullshit.** In pitches, in blog posts, in changelogs: say what is true. "We had a bug that caused data loss for 3 users, here's what happened and here's what we changed" builds more trust than any marketing copy.
