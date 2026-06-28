---
type: Design Doc
title: "Conductor Billing System Design"
description: Three-component billing architecture for Conductor: Stripe subscription plans, a credit system for execution and Bedrock token quota, and monthly AWS Bedrock pass-through reconciliation. Includes data model, budget enforcement, and dashboard data specifications.
resource: /startup/06-design/billing-design.md
tags: [billing, stripe, bedrock, credits, metered-billing, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor Billing System Design

## 1. Overview

Conductor billing has three distinct components that work together:

1. **Stripe Subscription** — monthly recurring charge for the tenant's plan tier (Builder $29/mo, Team $99/seat/mo).
2. **Credit System** — a bucket of IR execution quota and Bedrock token quota. Credits are included with each plan and purchasable as add-on packs. Enforced at runtime before each LLM_CALL.
3. **Bedrock Pass-Through Reconciliation** — Conductor calls AWS Bedrock on behalf of tenants from Conductor's AWS account. Monthly, Conductor reconciles actual Bedrock costs per tenant from CloudWatch and creates Stripe invoice line items — billing tenants at Bedrock list prices with zero markup.

The key design principle: **credits are enforced in code before every LLM_CALL**. A tenant cannot accidentally run up an unbounded bill. The only exception is Tier 2/3 GPU costs, which are included in the plan price (no per-token charge to tenants for private GPU usage).

---

## 2. Plan Tiers and Credit System

**1 credit = 1,000 Bedrock input tokens**

| Plan | Monthly Fee | IR Executions | Bedrock Credits Included | Model Access | Notes |
|---|---|---|---|---|---|
| Free | $0 | 500 | 0 | Tier 3 only (Gemma4-12B) | No Bedrock, no Tier 2 |
| Builder | $29/mo | 5,000 | 10,000 credits (~10M tokens) | Tier 1 + 2 + 3 | Solo developers |
| Team | $99/mo/seat | Unlimited | 50,000 credits/seat | Tier 1 + 2 + 3 | Per-seat pricing |
| Enterprise | Custom | Custom | Custom | All tiers + private VPC | Custom contract |

**Credit Add-On Packs (Stripe one-time charge)**

| Pack | Credits | Price | Effective Rate |
|---|---|---|---|
| Starter | 10,000 | $10 | $1.00/1k credits ($1.00/1M tokens) |
| Growth | 100,000 | $80 | $0.80/1k credits ($0.80/1M tokens) |
| Scale | 1,000,000 | $600 | $0.60/1k credits ($0.60/1M tokens) |

Credits do not expire within the current billing cycle. Unused credits roll over for up to 90 days.

**IR Execution Counting**
An "IR execution" is one complete invocation of a `pipeline_runs` row from `pending` to a terminal state (`completed`, `failed`, `aborted`). Retried steps within a run do not count as additional executions. Free tier: 500/month. When the monthly limit is hit, new run attempts are rejected with HTTP 402 and a clear message directing the user to upgrade.

---

## 3. Stripe Integration

**Products and Prices**

```
Stripe Products:
  conductor_builder  → Price: $29/mo recurring
  conductor_team     → Price: $99/mo/seat recurring
  conductor_enterprise → Price: custom (invoiced)

Stripe Metered Items (on each subscription):
  bedrock_credits    → unit: "credit", aggregation: SUM, billing period: monthly
```

**Usage Record Posting**
After each successful `LLM_CALL` on Bedrock:

```python
async def post_usage_record(tenant_id: UUID, credits_consumed: int, run_id: UUID) -> None:
    tenant = await db.get(Tenant, tenant_id)
    stripe.SubscriptionItem.create_usage_record(
        tenant.stripe_subscription_item_id_credits,
        quantity=credits_consumed,
        timestamp=int(time.time()),
        action="increment",
        idempotency_key=f"usage_{run_id}_{ir_index}",
    )
    # Also deduct from local credit balance for real-time enforcement
    await db.execute(
        update(Tenant)
        .where(Tenant.id == tenant_id)
        .values(credits_balance=Tenant.credits_balance - credits_consumed)
    )
```

Note: Stripe usage records are the source of truth for invoicing. The local `credits_balance` column is a cached counter for fast runtime enforcement — it is reconciled with Stripe at the start of each billing period.

**Webhook Events**

| Stripe Event | Handler Action |
|---|---|
| `customer.subscription.updated` | Update `tenants.tier`, reset credit limits, update `monthly_execution_limit` |
| `invoice.paid` | Reset `credits_balance` to plan's included credits, reset `monthly_execution_count` to 0 |
| `invoice.payment_failed` | Downgrade tenant to `suspended` status; reject new runs with HTTP 402 |
| `checkout.session.completed` | For credit pack purchases: add credits to `tenants.credits_balance` |
| `customer.subscription.deleted` | Downgrade to free tier immediately |

**Stripe Credit Pack Flow**
```
User clicks "Buy 100k credits" in dashboard
    │
    ▼
POST /billing/buy-credits { pack: "growth" }
    │
    ▼
Create Stripe Checkout Session (one-time payment, $80)
    │
    ▼
User completes payment on Stripe-hosted page
    │
    ▼
Stripe webhook: checkout.session.completed
    │
    ▼
Handler: UPDATE tenants SET credits_balance = credits_balance + 100000
         INSERT INTO billing_usage (event: "credit_purchase", credits: 100000)
```

---

## 4. Bedrock Pass-Through Reconciliation

Conductor calls Bedrock from its own AWS account (one account per environment: `conductor-prod`). All Bedrock API calls are tagged with `tenant_id` using the `RequestMetadata` field in the Bedrock `InvokeModel` API:

```python
boto3_client.invoke_model(
    modelId=model_id,
    body=json.dumps(request_body),
    contentType="application/json",
    accept="application/json",
    # RequestMetadata is logged to CloudWatch Logs Insights, not Bedrock billing API
    # Instead, we use our own model_calls table for per-tenant attribution
)
```

Since Bedrock does not support per-API-call cost tagging in CloudWatch (it reports aggregate model usage), Conductor uses its **own `model_calls` table** as the source of truth for per-tenant Bedrock attribution.

**Monthly Reconciliation Job (Lambda, runs on 2nd of each month)**

```python
async def reconcile_bedrock(billing_period: date) -> None:
    period_start = billing_period.replace(day=1)
    period_end = (period_start + timedelta(days=32)).replace(day=1)

    # 1. Query model_calls table for Bedrock usage per tenant in period
    usage_rows = await db.execute("""
        SELECT tenant_id,
               SUM(prompt_tokens) AS total_input,
               SUM(completion_tokens) AS total_output,
               model_name,
               COUNT(*) AS call_count
        FROM model_calls
        WHERE provider = 'bedrock'
          AND created_at >= :start AND created_at < :end
        GROUP BY tenant_id, model_name
    """, {"start": period_start, "end": period_end})

    for row in usage_rows:
        # 2. Calculate cost at Bedrock list prices
        cost_usd = calculate_bedrock_cost(row.model_name, row.total_input, row.total_output)

        # 3. Create Stripe invoice item for the tenant
        tenant = await db.get(Tenant, row.tenant_id)
        if tenant.stripe_customer_id and cost_usd > 0.01:  # skip negligible amounts
            stripe.InvoiceItem.create(
                customer=tenant.stripe_customer_id,
                amount=int(cost_usd * 100),  # cents
                currency="usd",
                description=f"Bedrock {row.model_name} usage: {row.total_input:,} input + {row.total_output:,} output tokens",
                period={"start": int(period_start.timestamp()), "end": int(period_end.timestamp())},
                idempotency_key=f"bedrock_reconcile_{row.tenant_id}_{billing_period.isoformat()}_{row.model_name}",
            )

        # 4. Record in billing_usage
        await db.execute(insert(BillingUsage).values(
            tenant_id=row.tenant_id,
            period_start=period_start,
            provider="bedrock",
            input_tokens=row.total_input,
            output_tokens=row.total_output,
            cost_usd=cost_usd,
        ))
```

**Bedrock List Prices (at time of writing; update monthly)**

| Model | Input $/1M tokens | Output $/1M tokens |
|---|---|---|
| Claude 3.5 Sonnet | $3.00 | $15.00 |
| Claude 3 Haiku | $0.25 | $1.25 |
| Llama 3 70B Instruct | $0.99 | $0.99 |
| Mistral Large | $4.00 | $12.00 |

---

## 5. Usage Metering Data Model

```sql
CREATE TABLE billing_usage (
  id                     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id              UUID NOT NULL REFERENCES tenants(id),
  run_id                 UUID REFERENCES pipeline_runs(id) ON DELETE SET NULL,
  period_start           DATE NOT NULL,
  provider               VARCHAR(50) NOT NULL,   -- bedrock | openai | vllm | system
  event_type             VARCHAR(50) NOT NULL DEFAULT 'llm_call',
  -- llm_call | credit_purchase | reconciliation | execution_count
  input_tokens           BIGINT NOT NULL DEFAULT 0,
  output_tokens          BIGINT NOT NULL DEFAULT 0,
  credits_consumed       INTEGER NOT NULL DEFAULT 0,
  cost_usd               DECIMAL(12,6) NOT NULL DEFAULT 0,
  stripe_usage_record_id VARCHAR(255),
  stripe_invoice_item_id VARCHAR(255),
  created_at             TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

The `model_calls` table (defined in data-model.md) is the detailed record of every LLM invocation. `billing_usage` is the billing-period aggregate and Stripe integration record. The two are related: `billing_usage.run_id` lets you join back to `model_calls` for line-item detail.

---

## 6. Budget Enforcement

Budget enforcement happens at **two levels**, both enforced in code (not in a prompt):

**Level 1 — Per-Run Context Budget**
Declared as `context budget: 32k tokens` in the pipeline source. The Context Manager (runtime component) counts tokens with tiktoken before assembling the messages array for each `LLM_CALL`. If the assembled context exceeds `budget_tokens`, it runs a summarization pass to compress the oldest context stack entry. If the context still exceeds the budget after summarization, the `LLM_CALL` is rejected with error E015.

**Level 2 — Per-Account Credit Limit**
Before each `LLM_CALL` that routes to Bedrock (Tier 1):
```python
async def check_budget(tenant_id: UUID, estimated_tokens: int) -> None:
    tenant = await db.get(Tenant, tenant_id)
    credits_needed = math.ceil(estimated_tokens / 1000)   # 1 credit = 1,000 tokens
    if tenant.credits_balance < credits_needed:
        raise BudgetExceededError(
            f"Insufficient credits: need {credits_needed}, have {tenant.credits_balance}. "
            f"Purchase credits at https://conductor.dev/billing"
        )
```

This check uses the `credits_balance` column (local cached counter, decremented after each LLM_CALL) for speed. If `credits_balance` goes negative (race condition on concurrent calls), the nightly reconciliation job detects and alerts on the discrepancy; the tenant's next invoice includes the overage.

**Free Tier Enforcement**
Free tier tenants have `monthly_execution_limit = 500` and `credits_balance = 0`. The LLM_CALL handler skips the credit check for Tier 3 (GPU) calls (no Bedrock cost) but increments `monthly_execution_count`. When `monthly_execution_count >= monthly_execution_limit`, new runs are rejected at the `POST /runs` endpoint with HTTP 402.

---

## 7. Dashboard Data

The billing dashboard combines Stripe API data and internal Postgres queries.

**Current Month Summary (refreshed every 5 minutes via materialized view)**

```sql
-- For a given tenant in the current billing period:
SELECT
  (SELECT monthly_execution_count FROM tenants WHERE id = $tenant_id) AS executions_used,
  (SELECT monthly_execution_limit FROM tenants WHERE id = $tenant_id) AS executions_limit,
  (SELECT credits_balance FROM tenants WHERE id = $tenant_id) AS credits_remaining,
  COALESCE(SUM(cost_usd), 0) AS bedrock_spend_usd
FROM billing_usage
WHERE tenant_id = $tenant_id
  AND period_start = DATE_TRUNC('month', CURRENT_DATE)
  AND provider = 'bedrock';
```

**Per-Pipeline Cost Breakdown**

```sql
SELECT
  p.name AS pipeline_name,
  COUNT(DISTINCT pr.id) AS run_count,
  AVG(pr.total_cost_usd) AS avg_cost_usd,
  MIN(pr.total_cost_usd) AS min_cost_usd,
  MAX(pr.total_cost_usd) AS max_cost_usd,
  SUM(pr.total_tokens) AS total_tokens
FROM pipeline_runs pr
JOIN pipelines p ON p.id = pr.pipeline_id
WHERE pr.tenant_id = $tenant_id
  AND pr.created_at >= DATE_TRUNC('month', CURRENT_DATE)
  AND pr.status = 'completed'
GROUP BY p.name
ORDER BY SUM(pr.total_cost_usd) DESC
LIMIT 10;
```

**Projected Monthly Spend**

```python
def project_monthly_spend(tenant_id: UUID, days_elapsed: int) -> Decimal:
    """Linear projection based on current month's spend rate."""
    current_month_spend = get_current_month_bedrock_spend(tenant_id)
    daily_rate = current_month_spend / max(days_elapsed, 1)
    return daily_rate * 30  # project to full 30-day month
```

**Key Dashboard Widgets**

| Widget | Data Source | Refresh |
|---|---|---|
| Executions used / limit | `tenants` table | Real-time |
| Credits remaining | `tenants.credits_balance` | Real-time |
| Bedrock spend this month | `billing_usage` aggregate | 5 minutes |
| Top 5 most expensive pipelines | `pipeline_runs` join | 5 minutes |
| Projected monthly spend | Computed from daily rate | 1 hour |
| GPU tok/s (if on paid plan) | CloudWatch metric | 1 minute |
| Next invoice date + amount | Stripe API | 1 hour |
