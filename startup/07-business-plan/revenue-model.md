---
type: Business Plan Section
title: "Conductor Labs — Revenue Model"
description: Pricing architecture, credit system mechanics, unit economics with gross margin calculations, and month-by-month Year 1 MRR projections for Conductor Labs' four-tier subscription model.
tags: [revenue, pricing, unit-economics, mrr, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Revenue Model — Conductor Labs

## Pricing Architecture

Four subscription tiers with a usage-credit overlay:

| Tier | Monthly Price | Included Executions | Overage | Target Buyer |
|------|--------------|--------------------|---------|--------------|
| **Free** | $0 | 500 | Not available | Evaluating developers |
| **Builder** | $29/mo | 5,000 | $0.01/execution | Individual devs shipping pipelines |
| **Team** | $99/seat/mo | 15,000/seat | $0.008/execution | Engineering teams |
| **Enterprise** | $2,000+/mo | Custom | Custom | Orgs needing SSO, VPC, SLAs |

An "execution" is one complete pipeline run from IR start to HALT. A single run may contain dozens of LLM_CALL and CALL instructions — the execution count is not per-instruction.

**What is NOT charged per-execution:**
- Bedrock token usage: billed at cost (pass-through, zero markup). Conductor earns no margin on Bedrock tokens.
- Gemma4 GPU tier: covered by subscription on Builder+. Free tier capped at 500 executions/mo total regardless of model tier.
- Compilation: always free. Type-checking, error diagnostics, and IR generation are not metered.

---

## Credit System Mechanics

Credits are the internal accounting unit. One credit = one pipeline execution.

**Credit allocation on subscription renewal (monthly):**
```
Free:       500 credits  (non-rollover — reset monthly)
Builder:  5,000 credits  (non-rollover — reset monthly)
Team:    15,000 credits per seat  (non-rollover — reset monthly)
Enterprise: configured at contract time  (may be annual rollover)
```

**Credit add-on packs (Builder+):**

| Pack | Price | Credits | Effective Rate |
|------|-------|---------|----------------|
| Starter | $10 | 1,000 | $0.010/exec |
| Growth | $40 | 5,000 | $0.008/exec |
| Scale | $150 | 25,000 | $0.006/exec |

Add-on credits expire 12 months from purchase. They are consumed after included credits are exhausted.

**Enforcement:**
- Budget check at run submission: if `credits_remaining < 1`, return HTTP 402 with upgrade CTA.
- Mid-run enforcement: runs that started with credits do not fail mid-execution if credits hit zero. The run completes; the overage is recorded and billed as a pay-as-you-go line item on next invoice (Builder+ only).
- Free tier: runs are killed at 500 executions/mo. No overage. Upgrade gate.

**Bedrock pass-through billing:**
Each `LLM_CALL` instruction that routes to Bedrock logs `input_tokens`, `output_tokens`, and `model_id` in the `model_calls` table. A nightly Lambda reconciles these against Bedrock's cost API and posts usage records to the tenant's Stripe subscription. The tenant is charged cost + 0% markup. This is intentional: token margin would compete with provider relationships we want as distribution partners.

---

## Unit Economics

### Builder Tier ($29/mo)

| Component | Value | Notes |
|-----------|-------|-------|
| MRR per subscriber | $29.00 | |
| Stripe fee | -$1.16 | 2.9% + $0.30 |
| Infrastructure per sub | -$5.50 | API gateway, compiler, runtime, Postgres, Redis, S3 amortized |
| GPU compute per sub | -$0.80 | ~500 Gemma4 inferences/mo average, $0.05/1M tokens effective |
| Support burden | -$0.80 | ~0.05 hrs/mo * $16/hr blended (async, community) |
| **Gross Margin** | **$20.74 (71.5%)** | |

At scale (1,000+ Builder subscribers), infra per subscriber drops to ~$2.00 through spot fleet amortization and Postgres connection pooling. Gross margin scales to ~80%.

### Team Tier ($99/seat/mo)

| Component | Value | Notes |
|-----------|-------|-------|
| MRR per seat | $99.00 | |
| Stripe fee | -$3.17 | |
| Infrastructure per seat | -$8.00 | Higher execution volumes, more Redis pub/sub |
| GPU compute per seat | -$2.00 | Teams run longer pipelines, more LLM calls |
| Support burden | -$3.00 | 0.15 hrs/mo shared (Slack, email) |
| **Gross Margin** | **$82.83 (83.7%)** | |

### Enterprise Tier ($2,000/mo floor)

| Component | Value | Notes |
|-----------|-------|-------|
| MRR | $2,000–$10,000 | Negotiated |
| Stripe fee | -$60 | Estimated, invoicing reduces |
| Infrastructure | -$300 | VPC deployment adds EKS cluster, dedicated RDS |
| Support SLA | -$200 | 4-hr response SLA requires async-on-call at this ARR |
| SOC2 amortized | -$100 | $30k/yr SOC2 audit / 25 enterprise customers |
| **Gross Margin** | **~$1,340 (67%)** | |

Enterprise margin is lower due to VPC and SLA costs but absolute contribution is highest.

---

## Month-by-Month Year 1 MRR Projection

**Assumptions:**
- Month 1–2: beta (free, 0 paying)
- Month 3: HN launch, first paying customers
- Free → Builder conversion: 4% of MAU
- Monthly MAU growth: 40% MoM through Month 8, decelerating to 15% MoM thereafter
- Churn: 6%/mo Builder (high early, improves), 3%/mo Team
- Team tier: ~5% of paying base by Month 9
- No Enterprise revenue in Year 1

| Month | MAU | Builder Subs | Team Seats | MRR | MoM Growth |
|-------|-----|-------------|-----------|-----|------------|
| 1 | 50 | 0 | 0 | $0 | — |
| 2 | 100 | 0 | 0 | $0 | — |
| 3 | 200 | 8 | 0 | $232 | — |
| 4 | 280 | 14 | 0 | $406 | +75% |
| 5 | 390 | 20 | 0 | $580 | +43% |
| 6 | 545 | 28 | 3 | $1,109 | +91% |
| 7 | 760 | 39 | 5 | $1,628 | +47% |
| 8 | 1,064 | 54 | 8 | $2,358 | +45% |
| 9 | 1,225 | 63 | 12 | $3,027 | +28% |
| 10 | 1,409 | 72 | 17 | $3,771 | +25% |
| 11 | 1,620 | 84 | 22 | $4,614 | +22% |
| 12 | 1,863 | 96 | 28 | $5,250 | +14% |

**Year 1 ARR at Month 12**: ~$63,000

This is a conservative projection. If HN launch drives a spike (500+ stars in week 1), Month 3 MAU could be 500+ and the trajectory steepens significantly. The model uses the conservative base case.

---

## Pricing Rationale

**Why $29 (not $19 or $49)?**

- $29/mo is below the "feels like a significant expense" threshold for individual developers who expense tools on personal cards without approval.
- n8n is $20/mo (self-hosted) and $50/mo (cloud). Cursor is $20/mo. GitHub Copilot is $10/mo. At $29 we are clearly "pro tool" without being "team budget approval required."
- Van Westendorp price sensitivity testing (see [experiments.md](../04-validation/experiments.md)) targets this range.

**Why $99/seat for Team?**

- Comparable: Datadog starts at $15/host/mo (infrastructure) but $99/seat is normal for developer-centric SaaS. Linear is $8–$16/seat. Hex is $24/seat. We are above these because we are a compiler + runtime + hosting bundle, not just a UI.
- Enterprise pricing of $2k+/mo puts Team at a natural 20× ceiling before the "enterprise conversation" is necessary. This means teams up to ~20 seats ($1,980) should self-serve on Team before they consider Enterprise.

**Why free tier exists?**

Three reasons:
1. Distribution: free users are the top of the PLG funnel. They write `.agent` files that get committed to repos (marketing).
2. Research: free users find bugs that paying users haven't hit yet. They are QA.
3. Network: the more `.agent` files exist in the world, the more Conductor is entrenched as the default format.

Free tier is deliberately generous (500 executions = 500 full pipeline runs) but structurally limited: no Bedrock (Gemma4 free tier only), no team features, no CI/CD integration, no priority support.

---

## Template Marketplace (Year 2+)

Once 5,000 GitHub stars and a meaningful template ecosystem exist, launch a marketplace:

- Developers publish `.agent` templates (e.g., "Code Review Pipeline," "Documentation Generator," "Security Audit Workflow")
- Buyers browse, install in one click, customize
- Revenue split: 70% to template author, 30% to Conductor
- Both paid and free templates

The marketplace is a network effect flywheel: more templates → more reasons to try Conductor → more installs → more templates. The 30% take rate on a modest $10–$50/template with 1,000 templates × 10 sales each = $30,000–$150,000 annual marketplace revenue. Not the core revenue driver, but a compounding distribution moat.
