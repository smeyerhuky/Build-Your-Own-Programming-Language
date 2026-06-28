---
type: Business Plan Section
title: "Conductor Labs — Financial Model"
description: Three-scenario 3-year P&L for Conductor Labs (Bootstrap / Seed $500k / Series A $3M), break-even analysis, infrastructure cost scaling model, and detailed use of seed funds breakdown.
tags: [financial-model, pnl, break-even, runway, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Financial Model — Conductor Labs

## Break-Even Analysis

**Infra floor** (minimum monthly cost to keep the service running with 0 users): ~$1,000/mo

| Component | Monthly Cost | Notes |
|-----------|-------------|-------|
| RDS Postgres (db.t3.medium) | $60 | Single-AZ, Multi-AZ at $120 when needed |
| ElastiCache Redis (cache.t3.micro) | $25 | Single node for dev/early prod |
| ECS Fargate (compiler + runtime) | $120 | 2 tasks × 0.5 vCPU / 1GB, low traffic |
| S3 + CloudFront | $20 | Static assets, pipeline bundles |
| Route 53 + ACM | $5 | DNS + TLS |
| Clerk (auth) | $25 | Up to 1,000 MAU free, then $25+/mo |
| Stripe (billing) | $0 | No fixed fee; 2.9% + $0.30 per transaction |
| GitHub Actions (CI) | $0 | Free for public repos |
| Datadog (observability) | $50 | Agent infra plan |
| Domain + email | $15 | |
| Misc (backups, logs, etc.) | $30 | |
| **Monthly floor** | **~$350** | |

Wait — the floor is actually lower than $1,000. At scale with GPU:

| Component | Monthly Cost | Notes |
|-----------|-------------|-------|
| Base infra (above) | $350 | |
| g5.xlarge GPU (Tier 3, on-demand) | $400 | 1 instance on-demand for Tier 3 coverage |
| g5.2xlarge GPU (Tier 2 spot) | $300 | Spot pricing ~$0.45/hr, 28 hrs/day coverage |
| **Monthly floor (with GPU)** | **~$1,050** | |

**Break-even at $1,050/mo infra floor:**

With Builder gross margin of $20.74/subscriber after Stripe fees and infra:
- Break-even = $1,050 / $20.74 ≈ **51 Builder subscribers**

Realistically, a mix of tiers. Approximate break-even:
- 40 Builder ($29) + 5 Team seats ($99) = $1,160 + $495 = $1,655 MRR → covers $1,050 infra floor and $450 misc costs with margin.
- **~50–62 paying subscribers reaches cash-flow neutral on infra**.

This is a very achievable threshold. n8n reached it in 3 months from launch.

---

## Scenario A: Bootstrap (No Raise)

**Assumption**: Founder works part-time or has savings runway. No salary from company. Revenue covers infra from Month 4 onward.

| Period | ARR | Cumulative Cash Out | Net Cash Position |
|--------|-----|--------------------|-----------------|
| Month 3 | $2,784 | -$3,000 | -$3,000 |
| Month 6 | $13,308 | -$6,300 | +$7,008 |
| Month 9 | $36,324 | -$9,450 | +$26,874 |
| Month 12 | $63,000 | -$12,600 | +$50,400 |
| Year 2 | $180,000 | -$50,000 | +$130,000 |
| Year 3 | $400,000 | -$120,000 | +$280,000 |

**Verdict**: Viable if founder can survive 6–8 months on savings. Slow growth trajectory. Limits ability to hire DevRel at $1M ARR milestone. GPU tier stays limited.

---

## Scenario B: Seed Round ($500,000)

**Assumption**: Raise $500k at $4M pre-money. 18-month runway target.

### Year 1 P&L (Seed Scenario)

| Category | Q1 | Q2 | Q3 | Q4 | Full Year |
|----------|----|----|----|----|----------|
| **Revenue** | $1,500 | $15,000 | $45,000 | $95,000 | **$156,500** |
| COGS (infra, GPU, Stripe) | -$3,000 | -$4,500 | -$9,000 | -$19,000 | -$35,500 |
| **Gross Profit** | -$1,500 | $10,500 | $36,000 | $76,000 | **$121,000** |
| Founder salary | -$30,000 | -$30,000 | -$30,000 | -$30,000 | -$120,000 |
| Infrastructure extras | -$5,000 | -$5,000 | -$8,000 | -$10,000 | -$28,000 |
| DevRel / marketing | -$5,000 | -$10,000 | -$15,000 | -$20,000 | -$50,000 |
| Legal / compliance | -$20,000 | -$5,000 | -$10,000 | -$5,000 | -$40,000 |
| **Operating Loss** | -$61,500 | -$39,500 | -$27,000 | +$11,000 | **-$117,000** |
| **Cash Remaining** | $438,500 | $399,000 | $372,000 | $383,000 | |

Achieves cash-flow positive in Q4 Year 1. Ends Year 1 with $383k cash (including seed proceeds).

### Year 2 P&L (Seed Scenario)

| Category | Q1 | Q2 | Q3 | Q4 | Full Year |
|----------|----|----|----|----|----------|
| **Revenue** | $150,000 | $210,000 | $300,000 | $420,000 | **$1,080,000** |
| COGS | -$30,000 | -$42,000 | -$60,000 | -$84,000 | -$216,000 |
| **Gross Profit** | $120,000 | $168,000 | $240,000 | $336,000 | **$864,000** |
| Salaries (founder + DevRel hire Q3) | -$40,000 | -$40,000 | -$80,000 | -$80,000 | -$240,000 |
| Infrastructure extras | -$15,000 | -$20,000 | -$28,000 | -$38,000 | -$101,000 |
| DevRel / marketing | -$25,000 | -$30,000 | -$40,000 | -$50,000 | -$145,000 |
| Legal / SOC2 | -$10,000 | -$5,000 | -$5,000 | -$5,000 | -$25,000 |
| **Operating Profit** | $30,000 | $73,000 | $87,000 | $163,000 | **$353,000** |

Year 2 ARR: $1,080,000 (crossing $1M ARR milestone mid-Year 2). DevRel hired at $1M ARR (Q3 Year 2).

### Year 3 P&L (Seed Scenario)

| Category | Full Year |
|----------|-----------|
| **Revenue** | $3,240,000 |
| COGS (30% gross margin erosion from Enterprise infra) | -$810,000 |
| **Gross Profit** | $2,430,000 |
| Salaries (founder + DevRel + Infra Eng + Sales Eng) | -$720,000 |
| Infrastructure + GPU fleet | -$300,000 |
| Marketing + conferences | -$200,000 |
| Legal + SOC2 Type II | -$80,000 |
| **Operating Profit** | **$1,130,000** |

Year 3 ARR: $3.24M growing at ~3× YoY.

---

## Scenario C: Series A ($3,000,000)

**Assumption**: Raise $3M at ~$15M pre-money after demonstrating $500k ARR (Month 18 in Scenario B). Used to accelerate enterprise go-to-market, expand GPU capacity, and hire sales.

| Period | ARR | Team Size | Key Milestones |
|--------|-----|-----------|----------------|
| Year 1 | $156,000 | 1 (founder) | PLG launch, 500 GitHub stars, 50 paying |
| Year 2 | $1,080,000 | 2–3 (+ DevRel) | $1M ARR, AWS Marketplace, SOC2 Type I |
| Year 3 (post-A) | $5,000,000 | 6–8 | Enterprise sales motion, SOC2 Type II, $5M ARR |

Series A use of funds ($3M):
- Engineering hires × 2 (Infra + Fullstack): $480,000/yr
- Sales/enterprise hire × 1: $200,000/yr
- DevRel (already hired): $160,000/yr
- GPU infrastructure (multi-region): $600,000/yr
- Marketing + events + AWS Marketplace co-sell: $300,000/yr
- Legal + compliance (FedRAMP feasibility): $120,000/yr
- Buffer / CEO salary: $200,000/yr
- **Total annual burn**: ~$2,060,000 → 18 months runway at $3M

---

## Infrastructure Cost Scaling Model

| MAU | Monthly Infra | Per-User Cost | Key Scaling Events |
|-----|--------------|---------------|--------------------|
| 100 | $350 | $3.50 | Single-region, no GPU |
| 500 | $1,050 | $2.10 | Add GPU (Tier 3 g5.xlarge) |
| 2,000 | $2,800 | $1.40 | Add GPU (Tier 2 g5.2xlarge spot), RDS Multi-AZ |
| 5,000 | $6,000 | $1.20 | ECS auto-scaling, Redis cluster, 2 GPU nodes |
| 10,000 | $11,000 | $1.10 | Multi-region, NAT gateway, enhanced monitoring |
| 50,000 | $35,000 | $0.70 | Spot fleet, reserved instances, CDN tier |
| 100,000 | $60,000 | $0.60 | Reserved GPU, Savings Plans, global edge |

Infrastructure cost per user **decreases** as scale increases (spot fleet amortization, reserved instance discounts, Postgres connection pooling reducing per-connection cost). This is a favorable characteristic for SaaS gross margin improvement over time.

---

## Seed Use of Funds (Detailed)

**Total Raise**: $500,000  
**Runway Target**: 18 months  
**Monthly Burn at Raise**: ~$18,000 (Month 1 post-raise, conservative)

| Category | Amount | Monthly | Purpose |
|----------|--------|---------|---------|
| Founder salary | $150,000 | $8,333 | Replaces freelance income; enables full-time focus |
| Infrastructure + GPU | $100,000 | $5,556 | AWS G5 spot fleet, reserved Postgres, multi-AZ, CDN |
| DevRel + content | $100,000 | $5,556 | Conference presence (1–2/yr), content production, community |
| Legal + compliance | $50,000 | $2,778 | Entity formation, IP, SOC2 Type I audit, AWS Marketplace onboarding |
| Reserve | $100,000 | — | 6-month emergency buffer; also funds Series A diligence costs |
| **Total** | **$500,000** | **~$22,000** | |

**Note on legal**: SOC2 Type I is non-negotiable for Enterprise tier and AWS Marketplace. Estimated $20k audit + $10k tooling (Vanta). Budget front-loaded to Month 3–6 of raise.

**Note on DevRel**: DevRel budget is not headcount — it is content production (video, demo infrastructure, conference speaker fees, event sponsorships). The first DevRel hire is a person funded separately at the $1M ARR milestone from operating cash flow.

---

## Key Financial Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| HN launch underperforms | 6–12 month revenue shortfall | Fallback: Discord seeding + YouTube; lower but more predictable |
| GPU spot interruption drives churn | Increased Tier 2 COGS, user complaints | Multi-region spot fleet with automatic failover; checkpoint-resume |
| Bedrock pricing increases | Compressed Bedrock pass-through margin | Pass increases to users within 30 days per TOS; Gemma4 self-host alternative |
| Churn exceeds 8%/mo | MRR stalls below break-even | Monthly NPS + exit interviews; identify top 3 failure modes before Month 4 |
| SOC2 delayed | No Enterprise sales in Year 1 | Start SOC2 evidence collection Day 1; use Vanta for automated evidence |
| Competitor raises $10M | Pricing pressure | Provider neutrality moat cannot be copied by any single LLM provider |
