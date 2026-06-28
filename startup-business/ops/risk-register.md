---
type: Risk Register
title: "LangCraft — Risk Register"
description: Top 10 risks for LangCraft with likelihood, impact, mitigation plan, and owner. Updated quarterly.
tags: [risk, ops, risk-register, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Risk Register

*Updated: 2026-06-28. Review quarterly (or when a risk materialises).*

Likelihood and Impact are scored 1 (low) – 5 (high). Risk Score = Likelihood × Impact.

---

## Risk summary table

| # | Risk | Likelihood | Impact | Score | Owner |
|---|---|---|---|---|---|
| R-01 | LLM API costs exceed budget | 4 | 3 | 12 | Engineering |
| R-02 | Enterprise sales cycles too long for runway | 3 | 4 | 12 | CEO |
| R-03 | Cannot find a business co-founder | 3 | 5 | 15 | Clinton |
| R-04 | Packt Publishing IP conflict | 2 | 4 | 8 | Legal |
| R-05 | Cloud sandbox security breach | 2 | 5 | 10 | Engineering |
| R-06 | Clinton's academic commitments slow startup | 3 | 4 | 12 | Clinton |
| R-07 | Large incumbent enters the market | 2 | 4 | 8 | CEO |
| R-08 | JFlex or BYacc dependency breaks | 1 | 3 | 3 | Engineering |
| R-09 | Key hire fails to materialise | 3 | 3 | 9 | CEO |
| R-10 | Pre-seed round takes longer than 6 months | 3 | 4 | 12 | Clinton + CEO |

---

## Detailed risk profiles

### R-01 — LLM API costs exceed budget

**Likelihood:** 4 | **Impact:** 3 | **Score:** 12

**Description:** The LLM assistant is a core product feature. At scale, GPT-4o or Claude API costs at $0.01–$0.05 per query could significantly reduce gross margin, especially on the unlimited Pro tier.

**Worst case:** 10 000 Pro users × 50 queries/day × $0.03/query = $15 000/day in LLM costs. Current Pro pricing ($29/month) would not cover this.

**Mitigation:**
1. Cap Pro queries at "unlimited reasonable use" (e.g., 200 queries/day) in the ToS; enforce above 500.
2. Cache common responses: the top 100 error messages + explanations can be pre-computed and served without an API call.
3. Use cheaper models for routine requests (GPT-4o mini, Haiku) and premium models for complex queries.
4. Monitor cost-per-user weekly; price Pro accordingly.
5. Negotiate volume discounts with OpenAI/Anthropic once MRR > $20K.

**Trigger:** If LLM COGS exceeds 25 % of revenue for two consecutive months, convene a cost review.

---

### R-02 — Enterprise sales cycles too long for runway

**Likelihood:** 3 | **Impact:** 4 | **Score:** 12

**Description:** Enterprise deals (LTV $80K+) are critical for reaching $100K ARR, but they have 3–9 month sales cycles. If we burn pre-seed capital waiting for enterprise revenue, we may run out of runway before closing.

**Mitigation:**
1. Do not depend on enterprise revenue to survive. Pro subscriptions should cover cash flow by Month 9.
2. Use enterprise deals as "bonus" revenue, not as primary revenue targets.
3. Pursue enterprise pilots ($0 first 3 months) to shorten time-to-first-revenue.
4. Get letter-of-intent (LOI) or intent-to-renew documentation early in the sales cycle.

**Trigger:** If MRR is <$5K at Month 9 post-launch, pivot GTM to double down on individual/academic rather than waiting for enterprise.

---

### R-03 — Cannot find a business co-founder

**Likelihood:** 3 | **Impact:** 5 | **Score:** 15

**Description:** The CEO co-founder role is critical. Without commercial leadership, Clinton cannot focus on product and technology. Recruiting a co-founder with the right background (developer tools GTM, fundraising, B2B SaaS) at this stage is genuinely hard.

**Mitigation:**
1. Start recruiting immediately via Clinton's academic and OSS network.
2. Post to YC co-founder matching, Founders Network, and Indie Hackers.
3. Attend developer tools conferences (KubeCon, QCon) to meet potential co-founders.
4. Consider hiring a fractional CMO/GTM lead as a bridge while searching.
5. If no co-founder by Q4 2026, consider whether Clinton can execute a more capital-efficient solo-founder path.

**Trigger:** If no co-founder candidate is in final-stage conversations by August 31, 2026, escalate to Plan B (solo-founder + fractional leadership).

---

### R-04 — Packt Publishing IP conflict

**Likelihood:** 2 | **Impact:** 4 | **Score:** 8

**Description:** The Packt Publishing contract for the book may contain clauses that affect LangCraft's ability to use, commercialise, or reproduce content from the book. If there is a non-compete or exclusivity clause, LangCraft's education product could be in conflict.

**Mitigation:**
1. Review the Packt agreement with IP counsel before incorporation (Month 1 action).
2. Ensure all LangCraft course content is *original* — not reproduced from the book.
3. If there is a conflict, negotiate with Packt for a content licensing deal (revenue share) or a carve-out.
4. Document the clean separation between book content (Packt's) and LangCraft curriculum (LangCraft's).

**Trigger:** Legal review before any public content is published on the LangCraft platform.

---

### R-05 — Cloud sandbox security breach

**Likelihood:** 2 | **Impact:** 5 | **Score:** 10

**Description:** The cloud sandbox executes user-submitted J0 code. A sandbox escape or code injection could allow malicious users to access other users' data, run arbitrary commands on the server, or access LangCraft's infrastructure.

**Mitigation:**
1. Each compilation runs in an isolated Docker container with:
   - No network access
   - Read-only filesystem (except a temporary `/tmp` mount)
   - Hard CPU and memory limits
   - 10-second execution timeout
2. Container images are pre-built and immutable; no package installation at runtime.
3. Source code is validated (max file size, max line count) before being passed to the compiler.
4. Penetration test before public launch ($5K budget).
5. Bug bounty program post-launch.
6. Incident response plan documented before beta.

**Trigger:** Any reported sandbox escape is a P0 incident; immediately suspend sandbox and audit.

---

### R-06 — Clinton's academic commitments slow startup

**Likelihood:** 3 | **Impact:** 4 | **Score:** 12

**Description:** Clinton is a tenured professor with teaching, advising, and service responsibilities. His time is constrained until he takes a sabbatical. If the startup demands more time than he can give, product progress will stall.

**Mitigation:**
1. Clinton negotiates a sabbatical starting Q4 2026 (standard for tenured faculty; high probability of approval).
2. The founding engineer absorbs day-to-day engineering, freeing Clinton for architecture, curriculum, and fundraising.
3. CEO co-founder absorbs all commercial activities (sales, marketing, investor relations).
4. Clinton sets a hard boundary: no more than 50 % of his time before sabbatical; 100 % after.

**Trigger:** If Clinton cannot spend 50 % of his time on LangCraft by October 2026, review sabbatical timeline.

---

### R-07 — Large incumbent enters the market

**Likelihood:** 2 | **Impact:** 4 | **Score:** 8

**Description:** JetBrains, Microsoft (GitHub Copilot), or Google (Gemini) could launch an interactive compiler construction platform that competes directly with LangCraft.

**Mitigation:**
1. Our moat is the *curriculum* (Clinton's 20 years of pedagogical IP), not the tooling. An incumbent cannot easily replicate it.
2. Move fast: establish community and brand recognition before an incumbent can respond.
3. Partner with potential acquirers early (JetBrains Marketplace listing is a distribution channel, not a threat).
4. If acquired, that's a successful outcome.

**Trigger:** If a funded competitor announces a similar product, accelerate enterprise and academic roadmap to differentiate.

---

### R-08 — JFlex or BYacc dependency breaks

**Likelihood:** 1 | **Impact:** 3 | **Score:** 3

**Description:** JFlex and BYacc are mature, open-source tools with stable APIs. However, a major version break or abandonment could require significant rework of the sandbox.

**Mitigation:**
1. Pin specific versions of JFlex and BYacc in the sandbox container image.
2. Maintain local forks of both tools as a fallback.
3. The J0 compiler already runs on both Java (JFlex/BYacc) and Unicon (Uflex/Iyacc) — if one breaks, the other still works.

**Trigger:** Monitor JFlex and BYacc release notes; review quarterly.

---

### R-09 — Key hire fails to materialise

**Likelihood:** 3 | **Impact:** 3 | **Score:** 9

**Description:** The founding engineer or Head of DevRel role takes longer than expected to fill, delaying the launch timeline.

**Mitigation:**
1. Start recruiting immediately (don't wait for SAFE to close — recruiting takes 2–3 months).
2. Have a pipeline of 5+ candidates per role before making an offer.
3. If hire is delayed >2 months: use contractors to fill the gap while the search continues.
4. For the engineering role: Clinton can do more himself temporarily; DevRel is higher-risk since it's not Clinton's strength.

**Trigger:** If no founding engineer offer is extended by October 1, engage a technical recruiting firm.

---

### R-10 — Pre-seed round takes longer than 6 months

**Likelihood:** 3 | **Impact:** 4 | **Score:** 12

**Description:** Fundraising is hard and unpredictable. If the pre-seed takes 9–12 months instead of 3–6, the company runs on Clinton's personal savings and the product launch is delayed.

**Mitigation:**
1. Start fundraising conversations immediately (before formal round launch).
2. Maintain a list of 20+ warm investor prospects.
3. If round is not closed by December 2026: pursue NSF SBIR grant ($275K, 6-month cycle) as a bridge.
4. If grant is delayed: consider revenue-based financing against early Pro subscription revenue.
5. Clinton's personal cash runway: 6–9 months (reduced living expenses during sabbatical).

**Trigger:** If no SAFE is signed by October 31, 2026, pivot to grant/bootstrap path.
