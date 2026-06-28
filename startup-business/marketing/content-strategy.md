---
type: Content Strategy
title: "LangCraft — Content Strategy"
description: Content strategy for LangCraft covering blog, YouTube, GitHub, and newsletter — themes, cadence, ownership, and KPIs.
tags: [content, marketing, strategy, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Content Strategy

---

## Purpose

Content is our primary acquisition channel pre-launch and our retention channel post-launch. Because we sell to developers — who research thoroughly and distrust marketing — high-quality technical content is our most valuable marketing asset.

**Content goals:**
1. Attract developers searching for "how to build a programming language."
2. Build credibility and trust with Clinton's technical authority.
3. Drive free-tier signups from organic search, Hacker News, and developer newsletters.
4. Retain and upsell existing users through ongoing educational value.

---

## Content pillars

| Pillar | Description | Persona |
|---|---|---|
| **Compiler fundamentals** | "How does a lexer work?" "What is an AST?" Technical explanations written for experienced developers | Marcus, Yuki |
| **LangCraft tutorials** | "How to add a new keyword to J0 in 15 minutes" — hands-on, sandbox-linked tutorials | Marcus, Yuki |
| **Enterprise DSL patterns** | "How Stripe's API language works" "How to design a policy DSL" | Chen Wei |
| **Academic / teaching** | "How to run a compilers course with LangCraft" "5 exercises for teaching type checking" | Dr. Priya |
| **Language design** | "Why J0 uses static typing" "Tradeoffs in DSL syntax design" | All |

---

## Channels and cadence

### 1. Technical blog (langcraft.io/blog)

**Cadence:** 2 posts/month  
**Author:** Clinton Jeffery (primary); guest posts from community  
**Format:** Long-form (1 500–3 000 words); code examples; diagrams; sandbox embeds

**Planned posts (first 6 months):**

| Month | Title | Pillar |
|---|---|---|
| 1 | "How I built a complete compiler from scratch — and what I learned" | Compiler fundamentals |
| 1 | "Why I turned a textbook into a startup" | Origin story / credibility |
| 2 | "The J0 compiler pipeline explained in 10 diagrams" | Compiler fundamentals |
| 2 | "Adding a new keyword to J0 in 15 minutes" | LangCraft tutorial |
| 3 | "How Stripe, Hashicorp, and Google build internal DSLs" | Enterprise DSL patterns |
| 3 | "What is an AST and why does it matter?" | Compiler fundamentals |
| 4 | "How to teach compiler construction in 2026" | Academic / teaching |
| 4 | "The 5 most common mistakes beginners make building a lexer" | LangCraft tutorial |
| 5 | "Why I chose J0 over a Python-like syntax for a teaching language" | Language design |
| 5 | "Building a calculator DSL with LangCraft — from grammar to bytecode" | LangCraft tutorial |
| 6 | "LangCraft 1.0: what we shipped and what's next" | Launch / milestone |
| 6 | "How enterprise teams are using LangCraft to ship DSLs" | Enterprise DSL patterns |

**SEO targets:** "build your own programming language," "how to build a compiler," "learn compiler construction," "DSL toolchain," "J0 language tutorial"

---

### 2. YouTube

**Cadence:** 1 video/month (starting month 2)  
**Author:** Clinton Jeffery  
**Format:** 10–20 minute screencasts; demonstration-first; no slides

**Planned videos:**

| Month | Title | Length |
|---|---|---|
| 2 | "Build your first lexer in LangCraft (live demo)" | 15 min |
| 3 | "From source code to token stream — the J0 lexer explained" | 12 min |
| 4 | "Parsing: how a parser turns tokens into an AST" | 18 min |
| 5 | "Type checking: how compilers catch your mistakes" | 15 min |
| 6 | "End-to-end: J0 source → bytecode → running program" | 20 min |

**Distribution strategy:**
- Upload to YouTube with full transcript (for SEO).
- Clip into 60-second shorts for Twitter/X and LinkedIn.
- Embed in corresponding blog post.
- Pitch to developer YouTube channels (Fireship, ThePrimeagen, Theo) for potential reaction/review.

---

### 3. GitHub

**Cadence:** Ongoing; every commit, release, and discussion is content  
**Author:** Clinton Jeffery + community  
**Format:** README, CHANGELOG, GitHub Discussions, Wiki PRs

**Actions:**
- Pin a "Try LangCraft →" CTA prominently in the README.
- Write detailed release notes for every compiler version bump.
- Respond to every GitHub issue within 48 hours.
- Create a "good first issue" label and actively recruit contributors.
- Host quarterly "compiler challenge" in GitHub Discussions (solve a problem, share your approach).

**GitHub is our #1 trust signal.** A well-maintained, actively-responded-to repo converts sceptical developers far better than any ad.

---

### 4. Email newsletter

**Cadence:** Monthly (starting month 3)  
**Audience:** Free-tier users + email signups  
**Format:** 500–800 words; 1 new blog post; 1 sandbox tip; 1 community spotlight

**Goals:**
- Retain free-tier users who haven't converted.
- Surface upgrade prompts naturally (e.g., "This month we shipped Phase 6 exercises — available on Pro").
- Build a direct relationship that doesn't depend on algorithm changes.

---

### 5. Hacker News

**Cadence:** Strategic (3–4 posts per year)  
**Author:** Clinton Jeffery (authentic voice)  
**Format:** "Show HN," "Ask HN," and top-level comments in relevant threads

*See [Launch Plan](./launch-plan.md) for the launch-week HN strategy.*

---

## Content KPIs

| Channel | KPI | Target (6 months) |
|---|---|---|
| Blog | Organic search sessions/month | 5 000 |
| Blog | Email signups from blog | 500 |
| YouTube | Total views | 25 000 |
| YouTube | Subscribers | 1 000 |
| GitHub | Stars | 3 000 |
| Newsletter | Open rate | >35 % |
| Newsletter | Click rate | >8 % |

---

## What we will NOT create

- ❌ No generic "10 programming tips" content — we are technical experts; content must reflect that.
- ❌ No paid social content before PMF — organic only until we know what converts.
- ❌ No whitepapers or gated content — developers will not fill in forms for PDFs.
- ❌ No LinkedIn carousel posts — not our audience.
