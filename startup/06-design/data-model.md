---
type: Design Doc
title: "Conductor Postgres Data Model"
description: Complete multi-tenant Postgres schema for Conductor's control plane database, including full DDL for all tables, ASCII ER diagram, row-level security policies, key indexes, and zero-downtime migration strategy.
resource: /startup/06-design/data-model.md
tags: [database, postgres, schema, rls, data-model, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor Postgres Data Model

## 1. Overview

The control plane database is a **multi-tenant Postgres 15** instance (AWS RDS Multi-AZ). Every table carries a `tenant_id` column, and row-level security (RLS) policies enforce that a database session bound to one tenant cannot read, write, update, or delete another tenant's rows — even if the application SQL contains a bug.

The application sets the tenant context at the start of each request:
```sql
SET LOCAL app.tenant_id = '<tenant_uuid>';
```

RLS policies read this session variable. Service accounts that need cross-tenant access (billing reconciliation, admin dashboards) use `SECURITY DEFINER` functions that bypass RLS with explicit, audited logic.

All tables use `UUID PRIMARY KEY DEFAULT gen_random_uuid()` for primary keys. Timestamps are `TIMESTAMPTZ` (always stored in UTC). JSON blobs use `JSONB` (indexed, binary-stored). Large binary content (encrypted keys) uses `BYTEA`.

---

## 2. Schema ER Diagram

```
┌────────────────────────────────────────────────────────────────────────┐
│                         TENANT LAYER                                    │
│                                                                         │
│   ┌──────────────┐                                                      │
│   │   tenants    │                                                      │
│   │──────────────│                                                      │
│   │ id (PK)      │────────────────────────────────────────────────┐   │
│   │ name         │                                                  │   │
│   │ slug         │                                                  │   │
│   │ tier         │                                                  │   │
│   │ stripe_*     │                                                  │   │
│   │ credits_bal  │                                                  │   │
│   └──────┼───────┘                                                  │   │
│          │ 1:N                                                      │   │
└──────────│─────────────────────────────────────────────────────────│───┘
           │                                                          │
    ┌──────┴────────────────────────────────┐              │
    │                                                 │              │
    ▼                                                 ▼              │
┌──────────┐                                  ┌────────────────┐    │
│  users   │                                  │   pipelines    │    │
│──────────│                                  │────────────────│    │
│ id (PK)  │                                  │ id (PK)        │    │
│ tenant_id│                                  │ tenant_id      │    │
│ email    │                                  │ name           │    │
│ clerk_id │                                  │ source (text)  │    │
│ role     │                                  │ ir_json (JSONB)│    │
└──────────┘                                  │ created_by → users│ │
                                              └────────┼───────┘    │
                                                       │ 1:N        │
                                                       ▼            │
                                              ┌──────────────────┐  │
                                              │  pipeline_runs   │  │
                                              │──────────────────│  │
                                              │ id (PK)          │  │
                                              │ tenant_id        │◄─┘
                                              │ pipeline_id      │
                                              │ status           │
                                              │ ir_index         │
                                              │ ir_json (JSONB)  │
                                              │ variables_json   │
                                              │ context_stack_json│
                                              │ total_tokens     │
                                              │ total_cost_usd   │
                                              └────────┼─────────┘
                                                       │ 1:N (all four)
                          ┌────────────────────────┼──────────────────────┐
                          │                            │                      │
                          ▼                            ▼                      ▼
                  ┌──────────────┐           ┌──────────────┐       ┌──────────────┐
                  │  run_events  │           │ model_calls  │       │  tool_calls  │
                  │──────────────│           │──────────────│       │──────────────│
                  │ id (PK)      │           │ id (PK)      │       │ id (PK)      │
                  │ tenant_id    │           │ tenant_id    │       │ tenant_id    │
                  │ run_id       │           │ run_id       │       │ run_id       │
                  │ event_type   │           │ model_name   │       │ tool_name    │
                  │ data_json    │           │ provider     │       │ args_json    │
                  │ ir_index     │           │ prompt_tokens│       │ result_type  │
                  └──────────────┘           │ cost_usd     │       │ latency_ms   │
                                            │ idempot_key  │       │ idempot_key  │
                                            └──────────────┘       └──────────────┘

tenants ──< billing_usage
```

---

## 3. Table Definitions (Complete DDL)

```sql
-- ============================================================
-- TENANTS
-- ============================================================
CREATE TABLE tenants (
  id                       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name                     VARCHAR(255) NOT NULL,
  slug                     VARCHAR(63) UNIQUE NOT NULL,
  tier                     VARCHAR(20) NOT NULL DEFAULT 'free',
  -- free | builder | team | enterprise | suspended
  stripe_customer_id       VARCHAR(255),
  stripe_subscription_id   VARCHAR(255),
  stripe_subscription_item_id_credits VARCHAR(255),  -- for posting usage records
  api_key_encrypted        BYTEA,   -- per-tenant Bedrock/OpenAI keys, KMS-encrypted
  kms_key_id               VARCHAR(255),
  credits_balance          INTEGER NOT NULL DEFAULT 0,
  monthly_execution_count  INTEGER NOT NULL DEFAULT 0,
  monthly_execution_limit  INTEGER NOT NULL DEFAULT 500,
  bedrock_enabled          BOOLEAN NOT NULL DEFAULT FALSE,
  created_at               TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at               TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- USERS
-- ============================================================
CREATE TABLE users (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id     UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email         VARCHAR(255) NOT NULL,
  clerk_user_id VARCHAR(255) UNIQUE NOT NULL,
  role          VARCHAR(20) NOT NULL DEFAULT 'developer',
  -- admin | developer | viewer
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(tenant_id, email)
);

-- ============================================================
-- PIPELINES
-- ============================================================
CREATE TABLE pipelines (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id    UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  name         VARCHAR(255) NOT NULL,
  description  TEXT,
  source       TEXT NOT NULL,     -- .agent source code (UTF-8)
  ir_json      JSONB,             -- last compiled IR (cache; invalidated on source edit)
  ir_version   INTEGER NOT NULL DEFAULT 0,  -- incremented on each successful compile
  tags         TEXT[],
  is_template  BOOLEAN NOT NULL DEFAULT FALSE,
  created_by   UUID NOT NULL REFERENCES users(id),
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(tenant_id, name)
);

-- ============================================================
-- PIPELINE RUNS
-- ============================================================
CREATE TABLE pipeline_runs (
  id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id            UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  pipeline_id          UUID REFERENCES pipelines(id) ON DELETE SET NULL,
  status               VARCHAR(20) NOT NULL DEFAULT 'pending',
  -- pending | running | paused | completed | failed | aborted
  ir_json              JSONB NOT NULL,
  ir_index             INTEGER NOT NULL DEFAULT 0,
  variables_json       JSONB NOT NULL DEFAULT '{}',
  context_stack_json   JSONB NOT NULL DEFAULT '[]',
  loop_state_json      JSONB NOT NULL DEFAULT '{}',  -- LOOP_START iterator state
  config_json          JSONB NOT NULL DEFAULT '{}',  -- timeout_s, max_iterations, etc.
  output               TEXT,
  error_json           JSONB,
  retry_count          INTEGER NOT NULL DEFAULT 0,
  total_tokens         INTEGER NOT NULL DEFAULT 0,
  total_cost_usd       DECIMAL(12,6) NOT NULL DEFAULT 0,
  idempotency_key      VARCHAR(255) UNIQUE,   -- prevents double-submission
  created_at           TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at           TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  completed_at         TIMESTAMPTZ
);

-- ============================================================
-- RUN EVENTS  (append-only event log; never UPDATE/DELETE)
-- ============================================================
CREATE TABLE run_events (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   UUID NOT NULL REFERENCES tenants(id),
  run_id      UUID NOT NULL REFERENCES pipeline_runs(id) ON DELETE CASCADE,
  event_type  VARCHAR(50) NOT NULL,
  -- step_started | llm_call | llm_response | tool_call | tool_result |
  -- checkpoint | step_completed | retrying | run_completed | run_failed
  data_json   JSONB NOT NULL,
  ir_index    INTEGER,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- MODEL CALLS  (one row per LLM API call, idempotency-keyed)
-- ============================================================
CREATE TABLE model_calls (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id         UUID NOT NULL REFERENCES tenants(id),
  run_id            UUID NOT NULL REFERENCES pipeline_runs(id) ON DELETE CASCADE,
  step_name         VARCHAR(255),
  model_name        VARCHAR(255) NOT NULL,
  provider          VARCHAR(50) NOT NULL,   -- bedrock | openai | vllm
  prompt_tokens     INTEGER NOT NULL,
  completion_tokens INTEGER NOT NULL,
  cost_usd          DECIMAL(12,6) NOT NULL,
  credits_consumed  INTEGER NOT NULL DEFAULT 0,
  latency_ms        INTEGER,
  idempotency_key   VARCHAR(255) UNIQUE NOT NULL,
  created_at        TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- TOOL CALLS  (one row per tool invocation, idempotency-keyed)
-- ============================================================
CREATE TABLE tool_calls (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL REFERENCES tenants(id),
  run_id          UUID NOT NULL REFERENCES pipeline_runs(id) ON DELETE CASCADE,
  step_name       VARCHAR(255),
  tool_name       VARCHAR(100) NOT NULL,
  args_json       JSONB,
  result_type     VARCHAR(100),
  latency_ms      INTEGER,
  exit_code       INTEGER,     -- for shell.run
  idempotency_key VARCHAR(255) UNIQUE NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- BILLING USAGE  (billing-period aggregates + Stripe integration)
-- ============================================================
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

-- ============================================================
-- UPDATED_AT TRIGGER (applied to tenants, pipelines, pipeline_runs)
-- ============================================================
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tenants_updated_at BEFORE UPDATE ON tenants
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER pipelines_updated_at BEFORE UPDATE ON pipelines
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER pipeline_runs_updated_at BEFORE UPDATE ON pipeline_runs
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

---

## 4. Row-Level Security Policies

RLS is enabled on every tenant-scoped table. The application layer sets `app.tenant_id` before any query. Service accounts that need cross-tenant access (billing reconciliation Lambda, admin API) use `SET ROLE service_account` with a role that has `BYPASSRLS`.

```sql
-- Enable RLS on all tenant-scoped tables
ALTER TABLE users             ENABLE ROW LEVEL SECURITY;
ALTER TABLE pipelines         ENABLE ROW LEVEL SECURITY;
ALTER TABLE pipeline_runs     ENABLE ROW LEVEL SECURITY;
ALTER TABLE run_events        ENABLE ROW LEVEL SECURITY;
ALTER TABLE model_calls       ENABLE ROW LEVEL SECURITY;
ALTER TABLE tool_calls        ENABLE ROW LEVEL SECURITY;
ALTER TABLE billing_usage     ENABLE ROW LEVEL SECURITY;

-- RLS policies: tenant can only see their own rows
CREATE POLICY tenant_isolation ON users
  AS PERMISSIVE FOR ALL
  TO application_role
  USING (tenant_id = current_setting('app.tenant_id', TRUE)::UUID);

CREATE POLICY tenant_isolation ON pipelines
  AS PERMISSIVE FOR ALL
  TO application_role
  USING (tenant_id = current_setting('app.tenant_id', TRUE)::UUID);

CREATE POLICY tenant_isolation ON pipeline_runs
  AS PERMISSIVE FOR ALL
  TO application_role
  USING (tenant_id = current_setting('app.tenant_id', TRUE)::UUID);

CREATE POLICY tenant_isolation ON run_events
  AS PERMISSIVE FOR ALL
  TO application_role
  USING (tenant_id = current_setting('app.tenant_id', TRUE)::UUID);

CREATE POLICY tenant_isolation ON model_calls
  AS PERMISSIVE FOR ALL
  TO application_role
  USING (tenant_id = current_setting('app.tenant_id', TRUE)::UUID);

CREATE POLICY tenant_isolation ON tool_calls
  AS PERMISSIVE FOR ALL
  TO application_role
  USING (tenant_id = current_setting('app.tenant_id', TRUE)::UUID);

CREATE POLICY tenant_isolation ON billing_usage
  AS PERMISSIVE FOR ALL
  TO application_role
  USING (tenant_id = current_setting('app.tenant_id', TRUE)::UUID);

-- Service account: BYPASSRLS role for cross-tenant operations
CREATE ROLE service_account BYPASSRLS LOGIN;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO service_account;

-- Application role: subject to RLS
CREATE ROLE application_role LOGIN;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO application_role;
```

**RLS Verification Query (run in CI)**
```sql
-- Test: tenant A cannot see tenant B's pipelines
SET LOCAL app.tenant_id = '<tenant_a_uuid>';
SELECT COUNT(*) FROM pipelines;  -- must return only tenant A's count

SET LOCAL app.tenant_id = '<tenant_b_uuid>';
SELECT COUNT(*) FROM pipelines;  -- must return only tenant B's count

-- Attempt to read across boundary (should return 0):
SET LOCAL app.tenant_id = '<tenant_a_uuid>';
SELECT COUNT(*) FROM pipelines WHERE tenant_id = '<tenant_b_uuid>';  -- expect 0
```

---

## 5. Key Indexes

```sql
-- pipeline_runs: most common query patterns
CREATE INDEX idx_pipeline_runs_tenant_status
  ON pipeline_runs(tenant_id, status);
  -- used by: GET /runs?status=running, RQ worker "find pending runs"

CREATE INDEX idx_pipeline_runs_tenant_pipeline
  ON pipeline_runs(tenant_id, pipeline_id, created_at DESC);
  -- used by: pipeline run history view

CREATE INDEX idx_pipeline_runs_idempotency
  ON pipeline_runs(idempotency_key)
  WHERE idempotency_key IS NOT NULL;
  -- used by: duplicate submission prevention

-- run_events: time-ordered log per run
CREATE INDEX idx_run_events_run_id
  ON run_events(run_id, created_at);
  -- used by: GET /runs/{id}/events (SSE replay)

-- model_calls: per-run cost aggregation and billing
CREATE INDEX idx_model_calls_run_id
  ON model_calls(run_id);
CREATE INDEX idx_model_calls_tenant_created
  ON model_calls(tenant_id, created_at)
  WHERE provider = 'bedrock';
  -- used by: monthly Bedrock reconciliation query

-- billing_usage: dashboard aggregation
CREATE INDEX idx_billing_usage_tenant_period
  ON billing_usage(tenant_id, period_start);

-- pipelines: tenant's pipeline list
CREATE INDEX idx_pipelines_tenant_updated
  ON pipelines(tenant_id, updated_at DESC);

-- users: Clerk auth lookup
CREATE INDEX idx_users_clerk_id
  ON users(clerk_user_id);
```

---

## 6. Migration Strategy

Migrations are managed by **Alembic** (Python). Migration files live in `migrations/versions/` and are version-controlled. Every migration must be reviewed and approved in a PR before merge.

**Migration File Naming**
```
migrations/versions/
  0001_initial_schema.py
  0002_add_loop_state_json.py
  0003_add_bedrock_enabled_flag.py
  ...
```

**Zero-Downtime Migration Protocol**
For any change to a high-traffic table (`pipeline_runs`, `model_calls`), use the expand/contract pattern:

```
Phase 1 — Expand (deploy with new code that writes both old + new columns):
  ALTER TABLE pipeline_runs ADD COLUMN loop_state_json JSONB;
  -- nullable; old rows get NULL; application writes to both columns

Phase 2 — Backfill (background job, not in a migration file):
  UPDATE pipeline_runs SET loop_state_json = '{}' WHERE loop_state_json IS NULL;

Phase 3 — Contract (deploy with code that reads only new column):
  ALTER TABLE pipeline_runs ALTER COLUMN loop_state_json SET NOT NULL SET DEFAULT '{}';

Phase 4 — (future sprint only) Remove old column if applicable
```

**Rules**
- Never `DROP COLUMN` in the same migration that removes references to it in application code. Always do DROP in the next sprint's migration, after confirming the old column is not read anywhere in production.
- Never add a `NOT NULL` column without a `DEFAULT` in a single migration on a table with data (causes a full-table rewrite with lock on Postgres < 11; use `DEFAULT` + backfill strategy above even on PG 15 for large tables).
- Always run `EXPLAIN ANALYZE` on new queries before adding indexes in production — index builds acquire `SHARE` locks.
- `CREATE INDEX CONCURRENTLY` for all new indexes on live tables (avoids write lock).

---

## 7. Tenant Isolation Verification

The automated isolation test suite runs in CI on every PR that touches the database layer.

**Test Fixture**
```python
@pytest.fixture
async def two_tenants(db):
    tenant_a = await create_tenant(db, name="Acme Corp", tier="builder")
    tenant_b = await create_tenant(db, name="Other Corp", tier="builder")
    pipeline_a = await create_pipeline(db, tenant_id=tenant_a.id, name="MyPipeline")
    pipeline_b = await create_pipeline(db, tenant_id=tenant_b.id, name="OtherPipeline")
    return tenant_a, tenant_b, pipeline_a, pipeline_b
```

**Isolation Tests**
```python
async def test_tenant_cannot_read_other_pipeline(two_tenants, db):
    tenant_a, tenant_b, pipeline_a, pipeline_b = two_tenants

    # Set session to tenant A
    await db.execute(text(f"SET LOCAL app.tenant_id = '{tenant_a.id}'"))
    results = await db.scalars(select(Pipeline))
    pipeline_ids = {r.id for r in results}

    assert pipeline_a.id in pipeline_ids
    assert pipeline_b.id not in pipeline_ids    # KEY ASSERTION

async def test_tenant_cannot_update_other_run(two_tenants, db):
    tenant_a, tenant_b, _, _ = two_tenants
    run_b = await create_run(db, tenant_id=tenant_b.id)

    await db.execute(text(f"SET LOCAL app.tenant_id = '{tenant_a.id}'"))
    # This UPDATE should silently affect 0 rows (RLS filters it out)
    result = await db.execute(
        update(PipelineRun).where(PipelineRun.id == run_b.id)
        .values(status="aborted")
    )
    assert result.rowcount == 0    # KEY ASSERTION: no cross-tenant mutation

async def test_service_account_bypasses_rls(two_tenants, service_db):
    _, _, pipeline_a, pipeline_b = two_tenants
    # service_db connection uses service_account role (BYPASSRLS)
    results = await service_db.scalars(select(Pipeline))
    pipeline_ids = {r.id for r in results}
    assert pipeline_a.id in pipeline_ids
    assert pipeline_b.id in pipeline_ids    # service account sees all
```

These tests run against a real Postgres instance (not SQLite) in CI using a `postgres:15` Docker container, because RLS is a Postgres-specific feature that cannot be replicated in SQLite or in-memory test databases.
