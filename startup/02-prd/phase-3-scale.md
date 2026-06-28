---
type: PRD Phase
title: "Phase 3 — Scale: GPU Serving + Enterprise + AWS Marketplace"
description: Phase 3 PRD for Conductor covering vLLM GPU infrastructure with Gemma4 and MTP speculative decoding, multi-region load balancing, enterprise SSO/SAML, VPC deployment, AWS Marketplace listing, pipeline governance, LangGraph export, fine-tuning service beta, and SLA monitoring. Exit gate is 3 enterprise LOIs and AWS Marketplace active.
resource: /startup/02-prd/overview.md
tags: [prd, phase-3, scale, vllm, gemma4, enterprise, aws-marketplace, sso, vpc, fine-tuning, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Phase 3 — Scale: GPU Serving + Enterprise + AWS Marketplace

## Phase Goal

Conductor becomes a viable enterprise product. The private GPU tier launches with Gemma4-26B running on vLLM with MTP speculative decoding, delivering <500ms p50 first-token latency and a privacy guarantee (code never leaves the Conductor GPU cluster). Enterprise buyers get SAML SSO, VPC deployment options, audit logs, and a pipeline governance workflow. The product lists on AWS Marketplace so enterprise customers can pay through their existing AWS billing relationship. The first enterprise deal closes.

## Phase Exit Gate

- 3 signed enterprise LOIs (Letter of Intent) for Team or Enterprise tier at $2k+/mo
- Gemma4-26B GPU tier live in at least 2 regions with p50 first-token latency <500ms (measured over 7 days)
- AWS Marketplace listing active (PAYG billing) with at least 1 paying Marketplace customer
- Enterprise SSO: Okta and Azure AD tested end-to-end with at least 1 real customer directory

## Architecture Decisions Locked in Phase 3

- **GPU instances**: AWS g5.xlarge spot instances (24GB VRAM, A10G GPU); 3× per region for HA
- **vLLM version**: v0.6.x (latest stable at Phase 3 start); OpenAI-compatible endpoint
- **Quantization**: AWQ (Activation-aware Weight Quantization); 4-bit, bfloat16 compute
- **Speculative decoding**: MTP (Multi-Token Prediction) draft model; 5 draft tokens; heuristic schedule
- **Model**: `google/gemma-4-26B-A4B-it` (26B, MoE 4B active); draft: `google/gemma-4-26B-A4B-it-assistant`
- **Fast tier model**: `google/gemma-4-12B-it` with Q4 quantization; single g5.xlarge per region
- **Regions**: us-east-1 (primary), eu-west-1, ap-southeast-1
- **Load balancing**: Cloudflare Load Balancing (latency-based routing) in front of regional ALBs
- **VPC deployment**: Terraform module targeting AWS; ECS Fargate for control plane, EC2 g5.xlarge for GPU nodes

---

## Tasks

### TASK-301: vLLM Serving Infrastructure
**Owner**: Coding Agent
**Complexity**: L
**Depends on**: TASK-109 (Postgres Schema — for usage tracking), TASK-209 (Model Router — extended in this phase)
**Acceptance criteria**:
- [ ] Terraform module at `infra/modules/vllm-cluster/` provisions per-region: 3× g5.xlarge spot instances in Auto Scaling Group with 1 on-demand fallback, Application Load Balancer, ECS task definition for vLLM container, EFS mount for model weights cache
- [ ] vLLM Docker image built from `infra/docker/vllm/Dockerfile`: base `vllm/vllm-openai:v0.6.x`, model pre-downloaded to EFS at deployment time (not pulled at cold start)
- [ ] vLLM startup command baked into ECS task definition:
  ```
  vllm serve google/gemma-4-26B-A4B-it \
    --speculative-model google/gemma-4-26B-A4B-it-assistant \
    --num-speculative-tokens 5 \
    --quantization awq \
    --dtype bfloat16 \
    --max-model-len 32768 \
    --served-model-name gemma4-26b-private \
    --api-key ${VLLM_API_KEY}
  ```
- [ ] `/health` endpoint on vLLM returns HTTP 200 when model is loaded; ECS health check fails (task replaced) if `/health` returns non-200 for 3 consecutive checks
- [ ] `model_download.py` script: downloads `google/gemma-4-26B-A4B-it` + AWQ weights from Hugging Face to EFS at `infra/scripts/model_download.py`; requires `HF_TOKEN` env var; idempotent (checks if weights already present)
- [ ] ALB routes `/v1/*` to vLLM; ALB access logs ship to S3 with 30-day retention
- [ ] Terraform outputs: `vllm_alb_dns`, `vllm_api_key_secret_arn`
- [ ] Infrastructure deployed to us-east-1 and eu-west-1; APAC added as stretch goal
- [ ] Load test: `locust` script at `infra/load_test/vllm_locust.py` sends 50 concurrent requests with 500-token prompts; p50 first-token latency asserted <500ms, p99 <2s

**Agent context needed**:
- vLLM 0.6.x docs (speculative decoding config, AWQ quantization, OpenAI-compatible server)
- AWS ECS Fargate docs (GPU task definitions require `EC2` launch type, not Fargate — use ECS EC2)
- Terraform AWS provider docs (autoscaling group, EFS)
- `conductor/runtime/router.py` (TASK-209 output — extend with vLLM routing)

**Deliverable**: `infra/modules/vllm-cluster/`, `infra/docker/vllm/Dockerfile`, `infra/scripts/model_download.py`, `infra/load_test/vllm_locust.py`

---

### TASK-302: Multi-Region Load Balancing
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-301 (vLLM Infrastructure — per-region ALBs must exist)
**Acceptance criteria**:
- [ ] Cloudflare Load Balancing configured via Terraform (`infra/modules/cloudflare-lb/`): origin pool per region (us-east-1, eu-west-1, ap-southeast-1); latency-based steering; health check on `/health` every 30s
- [ ] Cloudflare health check: monitors each region's vLLM ALB; marks origin unhealthy after 2 consecutive failures; routes traffic to next-nearest healthy region automatically
- [ ] Spot instance interruption handling: `infra/scripts/spot_interruption_handler.py` — AWS EventBridge rule triggers Lambda on `EC2 Spot Instance Interruption Warning`; Lambda calls ECS `deregisterContainerInstance` gracefully and increases on-demand ASG capacity by 1 before instance is terminated (2-minute window)
- [ ] Regional failover test: terminate all instances in us-east-1, assert that requests route to eu-west-1 within <60s (Cloudflare health check cycle + TTL)
- [ ] Latency metrics published to CloudWatch: `vllm.first_token_latency_ms` (p50, p95, p99) per region; dashboard at `infra/cloudwatch/vllm_dashboard.json`
- [ ] Terraform output: `conductor_vllm_endpoint` (the Cloudflare-fronted global endpoint)

**Agent context needed**:
- Cloudflare Load Balancing Terraform provider (`cloudflare/cloudflare`)
- AWS EventBridge + Lambda docs (spot interruption handling)
- `infra/modules/vllm-cluster/` (TASK-301 output)

**Deliverable**: `infra/modules/cloudflare-lb/`, `infra/scripts/spot_interruption_handler.py`, `infra/cloudwatch/vllm_dashboard.json`

---

### TASK-303: Tier 2 and Tier 3 Model Routing
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-301 (vLLM), TASK-302 (Load Balancing), TASK-209 (Model Router)
**Acceptance criteria**:
- [ ] `.agent` DSL model aliases extended: `model: gemma4-26b-private` routes to Tier 2 (vLLM, 26B model); `model: gemma4-12b-fast` routes to Tier 3 (vLLM, 12B model); existing `model: claude-3-5-sonnet` routes to Tier 1 (Bedrock)
- [ ] `conductor/runtime/router.py` updated: routing table maps alias → `(dispatcher_class, endpoint_config)`; `gemma4-*` aliases → `VLLMDispatcher` with endpoint from `CONDUCTOR_VLLM_ENDPOINT` env var
- [ ] `VLLMDispatcher` in `conductor/runtime/vllm_provider.py`: OpenAI-compatible client (`openai.AsyncOpenAI(base_url=vllm_endpoint)`); model name set to `gemma4-26b-private` or `gemma4-12b-fast` to match vLLM `--served-model-name`
- [ ] MTP acceptance rate surfaced in run metrics: vLLM returns `x-speculative-tokens-accepted` header per response; parsed and stored in `model_calls.speculative_tokens_accepted INT`
- [ ] Per-request token accounting: vLLM response includes `usage.prompt_tokens` and `usage.completion_tokens`; stored in `model_calls` table; cost computed using `conductor/runtime/pricing.py` (Tier 2 rate: $0.0003/1k tokens; Tier 3: free)
- [ ] Tier 2/3 access gated by plan: free tier → Tier 3 only; builder tier → Tier 2 + Tier 3 + Bedrock; enterprise tier → all tiers
- [ ] Tier routing decision logged in `pipeline_runs.model_tier` column for cost attribution
- [ ] 10-case test suite with mocked vLLM endpoint; 1 live integration test gated by `TEST_VLLM=true`

**Agent context needed**:
- `conductor/runtime/router.py`, `conductor/runtime/bedrock.py` (prior outputs)
- vLLM OpenAI-compatible server API docs (response format, speculative decoding headers)
- `conductor/runtime/pricing.py` (TASK-209 output)

**Deliverable**: `conductor/runtime/vllm_provider.py`, updated `conductor/runtime/router.py`, updated `conductor/runtime/pricing.py`, Alembic migration for `model_calls` and `pipeline_runs` schema changes

---

### TASK-304: Enterprise SSO (SAML/OIDC)
**Owner**: Coding Agent
**Complexity**: L
**Depends on**: TASK-114 (Auth + Tenant Setup), TASK-208 (Team Workspaces)
**Acceptance criteria**:
- [ ] SAML 2.0 SP-initiated SSO implemented via WorkOS (or Clerk's Enterprise Connections); Okta and Azure AD tested as IdPs
- [ ] OIDC supported as alternative to SAML: Google Workspace, Okta OIDC, Azure AD OIDC
- [ ] Just-in-time (JIT) user provisioning: on first SSO login from a new user in an enterprise tenant's domain, a `users` row is created automatically with the domain's default role (configurable per tenant: `admin_default_role`)
- [ ] RBAC roles: `admin` (full access), `developer` (create/run pipelines, cannot manage users), `viewer` (read-only: view runs and metrics, no create/run)
- [ ] Role assignment: configurable via IdP group-to-role mapping in `tenant_sso_config` table; example: Okta group `conductor-admins` → role `admin`
- [ ] Audit log table: `audit_log(id UUID PK, tenant_id UUID FK, user_id UUID, action TEXT, resource_type TEXT, resource_id UUID, details JSONB, created_at TIMESTAMPTZ)` — all writes (pipeline create/update/delete, run start/abort, user invite/remove, settings change) insert an audit log row
- [ ] `GET /audit-log` endpoint: admin-only, returns paginated audit log with filters by user, resource_type, date range
- [ ] Audit log export: `GET /audit-log/export?format=json&from=<date>&to=<date>` returns NDJSON for SIEM ingestion
- [ ] SSO setup UI in web IDE: `Settings > SSO` page for admins; inputs for IdP metadata URL (SAML) or client ID/secret (OIDC); validates connection by performing a test auth flow
- [ ] 15-case test suite: JIT provisioning, role enforcement, audit log entries on key actions, SSO config validation

**Agent context needed**:
- WorkOS SDK docs (SAML, OIDC, user provisioning) or Clerk Enterprise Connections docs
- `conductor/db/schema.sql` (TASK-109 output)
- `conductor/api/routes/team.py` (TASK-208 output)

**Deliverable**: `conductor/auth/sso.py`, `conductor/api/routes/sso.py`, `conductor/api/routes/audit.py`, Alembic migration for `audit_log` and `tenant_sso_config` tables, `web/src/SSOSettings.tsx`

---

### TASK-305: VPC Deployment Option
**Owner**: Coding Agent
**Complexity**: L
**Depends on**: TASK-301 (vLLM Infrastructure), TASK-109 (Postgres Schema)
**Acceptance criteria**:
- [ ] Terraform module at `infra/modules/vpc-deployment/` provisions a complete Conductor stack inside a customer-provided AWS VPC:
  - ECS cluster (EC2 launch type) for Conductor control plane (FastAPI, Redis, Postgres on RDS)
  - EC2 g5.xlarge ASG for vLLM GPU serving (Tier 2/3)
  - ALB for control plane API
  - VPC endpoints for Bedrock (if customer uses Tier 1)
  - IAM roles with least-privilege policies
  - CloudWatch Log Groups for all services
- [ ] Module inputs: `vpc_id`, `private_subnet_ids`, `public_subnet_ids`, `bedrock_enabled (bool)`, `vllm_enabled (bool)`, `db_instance_class`, `conductor_image_tag`
- [ ] Module outputs: `api_endpoint`, `rds_endpoint`, `vllm_endpoint`
- [ ] Customer brings their own Bedrock quota: the module configures Conductor's Bedrock dispatcher to use the customer's AWS account via VPC endpoints (no data transits Conductor's AWS account)
- [ ] Data isolation guarantee documented: in VPC mode, no pipeline source code, prompt content, or model responses are transmitted to Conductor Labs' infrastructure; only anonymous usage metrics (run count, token count) are sent to Conductor for billing via `POST /metering/usage` from within the VPC
- [ ] Deployment guide at `docs/vpc-deployment.md`: step-by-step from Terraform init to first pipeline run, including IAM setup for Bedrock access
- [ ] Terraform plan runs without errors against a real AWS account in CI (using `TF_VAR_` vars from GitHub Secrets)
- [ ] Upgrade path documented: `terraform apply` with updated `conductor_image_tag` performs rolling ECS deployment with zero downtime

**Agent context needed**:
- `infra/modules/vllm-cluster/` (TASK-301 output)
- Terraform AWS provider: VPC, RDS, ECS, IAM, VPC Endpoints
- AWS Bedrock VPC endpoint docs

**Deliverable**: `infra/modules/vpc-deployment/`, `docs/vpc-deployment.md`

---

### TASK-306: AWS Marketplace Listing
**Owner**: Both (Coding Agent builds the Marketplace-specific onboarding flow; Founder handles AWS Marketplace submission process)
**Complexity**: M
**Depends on**: TASK-110 (Stripe Billing), TASK-305 (VPC Deployment — for SaaS listing option)
**Acceptance criteria**:
- [ ] AWS Marketplace PAYG SaaS listing submission package created:
  - Product description, logo, screenshots (web IDE + metrics dashboard)
  - Pricing dimensions: `executions` (per 1k executions, $0.10), `bedrock_tokens` (per 1M tokens, $3.00), `seats` (per seat per month, $99)
  - `marketplace/entitlement_handler.py`: handles `aws-marketplace-entitlement` API webhook to sync Marketplace subscription events to `tenants` table
  - `marketplace/metering_handler.py`: posts hourly usage records to AWS Marketplace Metering Service (`boto3.client('meteringmarketplace').meter_usage(...)`)
- [ ] Marketplace-specific onboarding flow: when a user arrives from the Marketplace registration URL, Conductor creates a tenant with `billing_source: aws_marketplace`, `aws_customer_id` stored in `tenants` table; Stripe billing bypassed; all billing via AWS Metering
- [ ] ISV Accelerate program application submitted (founder action; tracked as task completion)
- [ ] AWS Marketplace `ResolveCustomer` API call implemented: on Marketplace registration redirect, Conductor backend calls `ResolveCustomer(RegistrationToken)` to validate and get `CustomerIdentifier`
- [ ] Test with AWS Marketplace sandbox: end-to-end registration flow tested; metering records appear in sandbox console
- [ ] `docs/marketplace-onboarding.md`: internal runbook for the registration → tenant creation → first run flow

**Agent context needed**:
- AWS Marketplace SaaS integration guide (ResolveCustomer, Entitlement, Metering APIs)
- `conductor/billing.py` (TASK-110 output — extend with Marketplace billing path)
- `conductor/db/schema.sql` (TASK-109 output — add `billing_source`, `aws_customer_id` columns)

**Deliverable**: `marketplace/entitlement_handler.py`, `marketplace/metering_handler.py`, `conductor/api/routes/marketplace.py`, `docs/marketplace-onboarding.md`, Alembic migration for `tenants` schema changes

---

### TASK-307: Enterprise Pipeline Governance
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-208 (Team Workspaces), TASK-304 (Enterprise SSO + Audit Log)
**Acceptance criteria**:
- [ ] Pipeline approval workflow: pipelines in enterprise tenants have a `governance_state` field: `draft`, `pending_approval`, `approved`, `rejected`; only `approved` pipelines can be run in production (enforced in `POST /runs` pre-check)
- [ ] `POST /pipelines/{id}/submit-for-approval` — transitions pipeline to `pending_approval`; notifies all `admin` users via email
- [ ] `POST /pipelines/{id}/approve` (admin only) — transitions to `approved`; `POST /pipelines/{id}/reject` (admin only) + `reason` body — transitions to `rejected` with reason stored
- [ ] Governance bypass for non-prod environments: `POST /runs` with `{env: "development"}` runs any pipeline regardless of governance state (rate-limited to 10 dev runs/day)
- [ ] Budget hard limits: `pipeline_budgets(pipeline_id UUID PK, max_tokens_per_run INT, max_cost_usd_per_run NUMERIC, max_runs_per_day INT)` table; `POST /pipelines/{id}/budget` (admin only) sets limits; limits enforced in `RunStateMachine` before each `LLM_CALL`
- [ ] Data classification tags: `pipeline_tags` table; tags: `public`, `internal`, `confidential`, `pii`; `confidential` and `pii` tagged pipelines: (a) log content stored encrypted at rest; (b) Tier 1 Bedrock disabled by default (can be re-enabled by admin)
- [ ] Compliance export: `GET /compliance/export?from=<date>&to=<date>` (admin only) returns a signed S3 URL to a ZIP containing: NDJSON of all `pipeline_runs` with metadata, NDJSON of all `model_calls` with token counts (no prompt/response content), NDJSON of `audit_log`
- [ ] 15-case test suite: governance state machine transitions, budget enforcement, tag-based Bedrock disable, compliance export structure

**Agent context needed**:
- `conductor/runtime/state_machine.py` (TASK-108 output)
- `conductor/api/routes/` (TASK-111, TASK-208 outputs)
- `conductor/audit_log.py` (TASK-304 output)

**Deliverable**: `conductor/api/routes/governance.py`, `conductor/api/routes/compliance.py`, Alembic migration for governance tables, updated `conductor/runtime/state_machine.py`

---

### TASK-308: LangGraph Export
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-106 (IR Emitter), TASK-202 (CLI)
**Acceptance criteria**:
- [ ] `conductor export --format langgraph <pipeline.agent> > pipeline_graph.py` CLI subcommand implemented in `conductor/export/langgraph.py`
- [ ] Export maps Conductor IR instructions to LangGraph constructs:
  - `LLM_CALL` → `langchain_core.runnables.RunnableLambda` wrapping `langchain_anthropic.ChatAnthropic` or `langchain_openai.ChatOpenAI` based on model alias
  - `CALL fs.*` / `CALL shell.run` → `@tool` decorated Python functions
  - `BRANCH` → `langgraph.graph.StateGraph.add_conditional_edges`
  - `LOOP_START / LOOP_END` → `StateGraph` node with a self-loop conditional edge
  - `CHECKPOINT` → `langchain_core.messages.HumanMessage` interrupt (LangGraph `interrupt_before`)
  - `CONTEXT_PUSH` → `ChatPromptTemplate.from_messages` with context messages prepended
- [ ] Output Python file is formatted with `ruff format` and passes `ruff check --select E,F`
- [ ] Output Python file includes a `if __name__ == "__main__": graph.invoke({})` block so it can be run directly
- [ ] Round-trip test: `conductor export --format langgraph refactor-dead-code.agent | python -c "import ast; ast.parse(open('/dev/stdin').read())"` (parses without syntax error)
- [ ] Migration report printed to stderr: list of Conductor features without direct LangGraph equivalent (e.g., `retrieve()` steps, `context budget:` declarations) with suggested workarounds
- [ ] 10-case test suite: 6 pipelines exported and their Python ASTs verified for key constructs; 4 pipelines with partial-export patterns verified to include `# TODO` stubs

**Agent context needed**:
- `conductor/ir.py`, `conductor/ir_emitter.py` (TASK-106 output)
- LangGraph `StateGraph` docs (node definition, conditional edges, interrupt_before)
- LangChain `ChatAnthropic`, `ChatOpenAI` docs
- `conductor/cli.py` (TASK-202 output)

**Deliverable**: `conductor/export/langgraph.py`, `tests/fixtures/export/`, `tests/test_export_langgraph.py`

---

### TASK-309: Fine-Tuning Service (Beta)
**Owner**: Coding Agent
**Complexity**: L
**Depends on**: TASK-301 (vLLM Infrastructure), TASK-304 (Enterprise SSO — fine-tuning is enterprise-only)
**Acceptance criteria**:
- [ ] `POST /fine-tuning/jobs` endpoint: accepts `{pipeline_id: str, step_name: str, examples: [{input: str, expected_output: str}]}` with at least 50 examples required; returns `{job_id: str, status: "queued"}`
- [ ] Fine-tuning job table: `fine_tuning_jobs(id UUID PK, tenant_id UUID FK, pipeline_id UUID FK, step_name TEXT, status TEXT, base_model TEXT, ft_model_id TEXT, examples_count INT, started_at TIMESTAMPTZ, completed_at TIMESTAMPTZ, error TEXT)`
- [ ] Fine-tuning backend: `conductor/fine_tuning/runner.py` runs `torchrun` with `axolotl` config on a spot g5.2xlarge (48GB VRAM for fine-tuning); base model `google/gemma-4-12B-it`; LoRA rank 16, alpha 32; 3 epochs; batch size 4 with gradient accumulation 4
- [ ] Fine-tuned model adapter uploaded to S3 at `s3://conductor-models/<tenant_id>/<job_id>/adapter/`; adapter merged with base model and uploaded to a per-tenant Hugging Face private repo or S3 path
- [ ] `vllm serve` updated to support loading per-tenant LoRA adapters: `--lora-modules tenant_ft:<s3_path>`; runtime routing: `model: ft:gemma4-12b:<job_id>` in `.agent` files routes to this adapter
- [ ] `GET /fine-tuning/jobs/{id}` returns status, loss curve (JSON array of `{step, loss}` pairs), estimated completion time
- [ ] Fine-tuning is enterprise-only: `POST /fine-tuning/jobs` returns HTTP 403 for non-enterprise tenants
- [ ] Beta label: all fine-tuning endpoints tagged with `X-Beta-Feature: true` header; docs clearly marked beta
- [ ] 10-case test suite: job creation, status polling, adapter S3 upload verified with mocked training run

**Agent context needed**:
- vLLM LoRA adapter docs (`--lora-modules` flag)
- Axolotl training framework docs (LoRA config YAML)
- AWS ECS GPU task definition docs (g5.2xlarge for training)
- `conductor/runtime/vllm_provider.py` (TASK-303 output)

**Deliverable**: `conductor/fine_tuning/runner.py`, `conductor/fine_tuning/axolotl_config.yaml.template`, `conductor/api/routes/fine_tuning.py`, Alembic migration for `fine_tuning_jobs`

---

### TASK-310: SLA Monitoring + On-Call
**Owner**: Both (Coding Agent sets up monitoring infrastructure; Founder configures PagerDuty and writes runbook)
**Complexity**: S
**Depends on**: TASK-302 (Multi-Region Load Balancing — for regional health checks)
**Acceptance criteria**:
- [ ] Better Uptime (or Checkly) external monitors configured for: `GET /health` on control plane (every 60s from 3 regions), vLLM `/health` per region (every 30s), web IDE root URL (every 60s)
- [ ] SLA targets documented in `docs/sla.md`: Enterprise tier 99.9% uptime (< 8.7 hours downtime/year); Builder tier 99.5%; Free tier best-effort
- [ ] PagerDuty service created; Better Uptime alert routes to PagerDuty on 2 consecutive failures; PagerDuty escalates to founder's phone after 5 minutes unacknowledged
- [ ] CloudWatch composite alarm: triggers if any of: (a) control plane `/health` fails, (b) vLLM p95 latency > 2s for 5 consecutive minutes, (c) Postgres connection pool exhausted (>95% connections used)
- [ ] Incident response runbook at `docs/incident-runbook.md`:
  - Severity definitions (P1: total outage, P2: degraded performance, P3: single-region failure)
  - Per-severity response SLA (P1: 15 min acknowledge, P2: 1 hour, P3: 4 hours)
  - Step-by-step diagnosis for: vLLM latency spike, spot instance interruption, Postgres connection exhaustion, Stripe webhook failure
  - Rollback procedure: `terraform apply -var conductor_image_tag=<previous>`
- [ ] Status page configured on Better Uptime or Statuspage.io: public URL at `status.conductorlabs.dev`; shows current status per component (Control Plane, GPU Tier, Bedrock Proxy, Web IDE)
- [ ] Monthly SLA report query: SQL in `docs/sla-report.sql` that computes uptime percentage from CloudWatch metric history

**Agent context needed**:
- Better Uptime API docs (monitor creation via API/Terraform)
- PagerDuty Terraform provider docs
- CloudWatch composite alarm Terraform docs
- `infra/modules/vllm-cluster/` (TASK-301 output — for CloudWatch metric names)

**Deliverable**: `infra/modules/monitoring/`, `docs/sla.md`, `docs/incident-runbook.md`, `docs/sla-report.sql`

---

## Phase 3 Risk Table

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| AWS g5.xlarge spot instances unavailable in target regions during product launch | High | High | Maintain fallback to g5.2xlarge on-demand; Terraform ASG configured with 2 instance type alternatives (`g5.xlarge`, `g5.2xlarge`, `g5.4xlarge`); use EC2 Spot Fleet with multiple instance types and AZs |
| Enterprise procurement timelines (security review, legal, MSA) extend Phase 3 by 2+ months | High | Medium | Begin enterprise prospect conversations in Phase 2 with design partner program; collect LOIs before full enterprise feature readiness; target developer-led adoption (bottom-up) to bypass procurement for initial deals |
| AWS Marketplace review process rejects listing due to missing security documentation | Medium | Medium | Engage AWS ISV Accelerate team in Phase 2 for pre-listing review; prepare SOC 2 Type I readiness documentation (not full certification, but evidence collection) as part of Phase 3 |
| Gemma4-26B MTP speculative decoding does not reach <500ms p50 on A10G with AWQ + 32k context | Medium | High | Run latency benchmarks in a dev vLLM cluster early in Phase 3; tune `--num-speculative-tokens` (try 3 and 8 as alternatives to 5); if target not met, shorten default context budget to 16k and document tradeoff |
| OpenAI or Anthropic announces a competing "workflow" product that makes Conductor redundant for simple use cases | Low | High | Conductor's value is the compiler (type safety, auditability, portability across providers) — not the models. Accelerate the portability-first story: publish benchmarks showing pipeline portability across Bedrock, OpenAI, and Gemma4 |
| vLLM version upgrade breaks AWQ quantization or MTP speculative decoding (has happened in 0.5.x → 0.6.x) | Medium | Medium | Pin vLLM version in Docker image; maintain a canary deployment (5% of traffic) on new vLLM versions before full rollout; automated latency regression test as part of deploy pipeline |

---

## Phase Exit Gate Verification

Before declaring Phase 3 complete, verify each criterion:

- [ ] Enterprise LOIs: 3 signed LOI documents on file in CRM (HubSpot or equivalent); each LOI specifies ARR > $2k/mo
- [ ] GPU tier latency: `SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY first_token_ms) FROM model_calls WHERE model LIKE 'gemma4-%' AND called_at > NOW() - INTERVAL '7 days'` returns < 500
- [ ] AWS Marketplace: Marketplace console shows listing status "Available"; at least 1 customer subscription with `billing_source = 'aws_marketplace'` in `tenants` table
- [ ] Enterprise SSO: Okta and Azure AD integration test reports (produced by TASK-304 test suite) both green; at least 1 real customer directory connected
- [ ] SLA monitoring: status page at `status.conductorlabs.dev` publicly accessible; PagerDuty test alert successfully pages founder's phone
- [ ] VPC deployment: at least 1 successful Terraform apply of `infra/modules/vpc-deployment/` in a test AWS account; deployment guide walkthrough completed without errors
