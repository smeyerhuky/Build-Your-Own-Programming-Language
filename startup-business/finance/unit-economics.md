---
type: Unit Economics
title: "LangCraft — Unit Economics"
description: Detailed CAC, LTV, gross margin, and payback period analysis by customer segment for LangCraft.
tags: [unit-economics, finance, cac, ltv, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Unit Economics

---

## Summary table

| Segment | ARPU/mo | Gross Margin | Monthly Churn | LTV | CAC | LTV:CAC | Payback |
|---|---|---|---|---|---|---|---|
| Pro (individual) | $29 | 72 % | 4.5 % | $464 | $18 | 25:1 | 0.9 mo |
| Academic (per student) | $17 | 78 % | 3 % (semester-end) | $340 | $5 | 68:1 | 0.3 mo |
| Team (5 seats) | $99 | 74 % | 3 % | $2 450 | $45 | 54:1 | 0.6 mo |
| Enterprise | $3 000 | 80 % | 8 % (annual) | $30 000 | $8 000 | 3.75:1 | 32 mo |

*LTV = ARPU × Gross Margin % / Monthly Churn Rate*

---

## Customer Acquisition Cost (CAC)

### How CAC is calculated

```
CAC = Total Sales & Marketing Spend in period
      ÷ New paying customers acquired in period
```

For PLG businesses, most acquisition cost is in content creation and infrastructure, not sales headcount.

### CAC by channel (estimated, Year 1)

| Channel | Spend/month | New customers/month | CAC |
|---|---|---|---|
| GitHub (organic) | $0 (Clinton's time) | 15 Pro | ~$0 |
| Blog / SEO | $500 (tools + time) | 20 Pro | $25 |
| Hacker News (launch) | $0 | 100 Pro (launch week) | $0 |
| YouTube | $300 (equipment, editing) | 10 Pro | $30 |
| Email nurture | $100 (Mailchimp) | 12 Pro | $8 |
| Academic outreach | $200 (conference, email) | 2 cohorts | $100/cohort |
| Enterprise (Sales Engineer) | $15 000/mo (when hired) | 1 enterprise | $15 000 |

**Blended CAC (Year 1, pre-enterprise-hire):** ~$18 per Pro customer  
**Blended CAC (Year 2, with Sales Engineer):** ~$40 (enterprise contracts pull the average up)

---

## Lifetime Value (LTV)

### Pro (individual)

```
ARPU/month         = $29
Gross margin       = 72 %
Monthly churn      = 4.5 %
LTV = $29 × 0.72 / 0.045 = $464
```

**Average Pro subscription length:** ~22 months (1 / 4.5 % churn)  
**Revenue over lifetime:** $29 × 22 = $638 gross; $459 net (after COGS)

**Factors that improve LTV:**
- Users who complete the full course churn at <2 %/month — 3× the lifetime
- Annual plan buyers ($249/yr) have ~1 % monthly churn equivalent — 5× LTV of monthly

---

### Academic cohort (per student)

```
ARPU/month         = $17 (cohort discount applied)
Gross margin       = 78 %
Avg cohort length  = 4 months (semester); some students renew annually
LTV (4-month cohort) = $17 × 0.78 × 4 = $53/student
LTV (if annual renewal) = $17 × 0.78 × 12 = $159/student
```

**The real unit is the instructor, not the student.** An instructor with 30 students generates:
- $17 × 30 students × 4 months = $2 040/cohort
- Renews 2× per year (fall and spring): $4 080/year
- Over 3 years (low churn on institutional relationships): $12 240 LTV per instructor

**Instructor CAC:** ~$200 (conference presence or direct outreach)  
**Instructor LTV:CAC:** ~61:1

---

### Enterprise

```
ARPU/month         = $3 000 (midpoint of $1 500–$6 000 range)
Gross margin       = 80 %
Annual churn       = 8 % (approximately 0.7 %/month)
LTV = $3 000 × 0.80 / 0.007 = $342 857
(Capped at practical limit of ~10 years: $3 000 × 0.80 × 120 = $288 000)
```

**Practical LTV (3-year average):** $3 000 × 0.80 × 36 months = $86 400 per enterprise customer

**Enterprise CAC:** $8 000 (Sales Engineer time: 3 months at 30 % of $180K salary + travel + tools)  
**Enterprise LTV:CAC (3-year):** 10.8:1  
**Enterprise payback:** 8 000 / ($3 000 × 0.80) = 3.3 months

---

## Gross margin by cost driver

| Cost driver | Variable or fixed | Impact on margin |
|---|---|---|
| LLM API (GPT-4o) | Variable (per query) | Largest COGS item; ~$0.01–$0.03 per query |
| Sandbox compute | Variable (per compilation) | ~$0.001–$0.005 per compilation |
| CDN/storage | Fixed-ish | Small; flat at scale |
| Stripe fees | Variable (2.9 % + $0.30) | ~3 % drag on revenue |

**LLM cost management strategy:**
- Free tier limited to 5 queries/day = low LLM cost per free user.
- Pro unlimited — but Pro users subsidise the cost through $29/mo revenue.
- At 500 Pro users × 10 queries/day × $0.02/query × 30 days = $3 000/month LLM cost.
- At $14 500 MRR from 500 Pro users, this is ~20 % COGS — acceptable but needs monitoring.
- As LLM prices decline (historically -40 %/year), margin improves automatically.

---

## Unit economics evolution

| Year | Blended LTV | Blended CAC | LTV:CAC | Key driver |
|---|---|---|---|---|
| Year 1 | $450 | $18 | 25:1 | PLG-only; no enterprise |
| Year 2 | $800 | $35 | 23:1 | Mix of PLG and early enterprise |
| Year 3 | $2 500 | $50 | 50:1 | Enterprise ACV lifts average LTV significantly |

*Enterprise contracts have high CAC but dramatically high LTV — the mix of segments is what matters.*
