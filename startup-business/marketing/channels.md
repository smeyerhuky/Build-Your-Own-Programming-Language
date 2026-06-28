---
type: Channel Scorecard
title: "LangCraft — Marketing Channels"
description: Evaluation and prioritisation of LangCraft's marketing channels with effort, reach, and conversion scores.
tags: [channels, marketing, distribution, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Marketing Channels

Each channel is scored on **reach** (how many qualified developers can we reach), **effort** (how much time/money it takes to activate), and **conversion** (how well it converts to signups). Score is 1–5 (higher = better).

---

## Channel scorecard

| Channel | Reach | Effort (lower = easier) | Conversion | Priority | Phase |
|---|---|---|---|---|---|
| GitHub (repo + README) | 5 | 1 | 5 | **P0** | Now |
| Hacker News | 5 | 2 | 4 | **P0** | Launch |
| Technical blog (SEO) | 4 | 3 | 4 | **P0** | Now |
| Email newsletter | 3 | 2 | 5 | **P0** | Month 3 |
| Reddit (r/ProgrammingLanguages, r/Compilers) | 3 | 2 | 3 | **P1** | Now |
| YouTube | 4 | 3 | 3 | **P1** | Month 2 |
| Twitter/X | 3 | 2 | 2 | **P1** | Now |
| Developer newsletters (TLDR, Pointer) | 4 | 2 | 3 | **P1** | Launch |
| LinkedIn | 2 | 2 | 2 | **P2** | Month 4 |
| Academic conferences (SIGCSE, PLDI) | 3 | 4 | 4 | **P1** | Month 6 |
| Paid search (Google Ads) | 3 | 3 | 2 | **P2** | After PMF |
| Podcast appearances | 3 | 3 | 3 | **P2** | Month 6 |
| Dev.to / Hashnode | 3 | 2 | 2 | **P2** | Month 3 |
| Product Hunt | 2 | 2 | 2 | **P2** | Launch |
| VS Code Marketplace | 4 | 2 | 4 | **P1** | When extension ships |

---

## Channel deep-dives

### GitHub — Priority P0

GitHub is our most important channel. Developers trust GitHub stars as a proxy for quality. The existing repository already has organic traction from the book.

**Actions:**
- Add a prominent "Try it on LangCraft →" CTA to the README (above the fold).
- Add a `[sandbox]` badge to every chapter folder linking to the corresponding LangCraft exercise.
- Maintain a detailed CHANGELOG — serious developers read it.
- Respond to every issue within 48 hours.
- Label issues `good first issue`, `help wanted` to invite contributors.

**Metrics:** Stars/week; forks/month; README click-through rate.

---

### Hacker News — Priority P0

HN is the highest-leverage single post for reaching senior developers. A front-page "Show HN" can drive 500–2 000 signups in 24 hours. Clinton's background (academic, author, compiler expert) is exactly the kind of credibility that HN readers respect.

**Tactics:**
- Launch "Show HN" post on day of public launch (see [Launch Plan](./launch-plan.md)).
- Participate authentically in threads about compiler construction, DSLs, and language design.
- Post an "Ask HN" on "best resources for learning compiler construction" to drive organic discovery.
- Never post marketing content — only genuine technical contributions.

**Metrics:** Points, comments, signups attributed to HN.

---

### Technical blog (SEO) — Priority P0

Organic search is the most durable acquisition channel. The target keywords have significant search volume and low-to-medium competition:

| Keyword | Est. monthly searches | Difficulty |
|---|---|---|
| "build your own programming language" | 2 400 | Medium |
| "how to build a compiler" | 8 100 | Medium |
| "learn compiler construction" | 1 300 | Low |
| "how to build a lexer" | 900 | Low |
| "DSL toolchain" | 600 | Low |
| "J0 language tutorial" | ~0 (brand new) | Very low |

**Goal:** Own the first page of Google for "build your own programming language" within 12 months of launch.

**Tactics:**
- Publish 2 long-form technical posts/month (see [Content Strategy](./content-strategy.md)).
- Optimise each post with target keywords, internal links to the sandbox, and a clear email capture CTA.
- Use structured data (article schema) on all posts.

---

### Email newsletter — Priority P0

Email is the highest-conversion channel for moving free users to paid. It is the only channel where we control the relationship directly (no algorithm).

**Tactics:**
- Capture email at signup (opt-in for newsletter).
- Monthly newsletter: 1 featured post + 1 sandbox tip + 1 community spotlight.
- Automated nurture sequence for new signups (5 emails over 14 days, each introducing a new concept and linking to a sandbox exercise).
- Upgrade email: fires when a user completes Phase 3 or hits the free tier LLM limit.

**Metrics:** Open rate (target >35 %); click rate (target >8 %); MRR attributed to email.

---

### Reddit — Priority P1

Reddit's `r/ProgrammingLanguages` (90K members), `r/Compilers` (35K members), and `r/learnprogramming` (6M+ members) are good audiences for organic content.

**Rules:**
- Never post promotional content directly. Participate in discussions.
- Post new blog posts as genuine contributions ("I wrote this because I kept seeing this question...").
- Respond to "how do I learn compiler construction" questions with the LangCraft free tier link + context.
- Do not mention LangCraft unless directly relevant to the thread.

---

### VS Code Marketplace — Priority P1 (when extension ships)

The VS Code Marketplace is an acquisition channel in its own right — developers browsing language support extensions will find the J0 extension.

**Actions:**
- Publish extension with a thorough README, screenshots, and link to langcraft.io.
- Add "also available as a web sandbox at langcraft.io" in the extension description.
- Respond to all marketplace reviews.

---

### LinkedIn — Priority P2

Lower priority for Phase 1 — LinkedIn reaches business buyers, not developers. More relevant when the enterprise tier is ready.

**When to activate:** After enterprise product launch (Q2 2027). Use for targeting VP Engineering, CTO, and Platform Engineering roles at companies known to build DSLs.

---

### Paid search — Priority P2

Do not run paid search until product-market fit is confirmed. CAC will be too high pre-PMF. Revisit when conversion rate from free → pro is >8 % and TTFB is <8 minutes.

**If/when activated:** Target branded keywords ("langcraft") + "build programming language tutorial" + "compiler construction course."
