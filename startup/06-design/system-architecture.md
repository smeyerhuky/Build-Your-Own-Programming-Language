---
type: Design Doc
title: "Conductor System Architecture"
description: Full system architecture for Conductor, a compiled DSL for agentic coding workflows. Covers component diagram, compile-to-run data flow, external integrations, scalability, security, and disaster recovery across all system layers.
resource: /startup/06-design/system-architecture.md
tags: [architecture, system-design, conductor, infrastructure, scalability]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor System Architecture

## 1. Architecture Overview

Conductor is a two-phase system: a **compiler service** that translates `.agent` source files into an IR instruction list, and a **runtime orchestrator** that walks that IR, dispatching LLM calls, tool invocations, and human checkpoints. The compiler is intentionally stateless — it receives source bytes and returns IR JSON — which means it scales horizontally on ECS Fargate with zero coordination between instances. The target latency is under 200ms at p50 for a typical 10-step pipeline. Because no session state lives in the compiler, a failed instance is simply replaced; no run data is lost.

The runtime orchestrator is stateful. Every IR instruction execution persists the updated run state (current `ir_index`, the variables map, the context stack, retry count) to Postgres before moving to the next instruction. This makes every run restartable from the last committed step — a property central to the reliability philosophy described in `harness/README.md`. Each active pipeline run executes in its own ECS task so that one runaway or long-running pipeline cannot starve others. The runtime communicates progress to clients via an SSE event stream, backed by a Redis pub/sub channel per `run_id`.

The system is multi-tenant from the ground up. Postgres enforces row-level security (RLS) so that a query issued in a tenant's application context cannot read another tenant's pipelines or runs, even if the SQL contains a bug. The GPU serving layer (vLLM on AWS g5.xlarge spot instances) exposes an OpenAI-compatible endpoint so the model router swaps only the base URL when routing between Bedrock, OpenAI, and the private GPU tier. Billing runs through Stripe metered billing for subscription charges and a monthly AWS CloudWatch reconciliation pass for Bedrock pass-through costs.

---

## 2. Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                   │
│  ┌───────────────────┐   ┌──────────────────────┐   ┌──────────────────┐   │
│  │   Web IDE         │   │  VS Code Extension   │   │  GitHub Actions  │   │
│  │  React + Monaco   │   │  + conductor CLI     │   │  conductor-action │  │
│  │  TailwindCSS      │   │  (.agent syntax hl)  │   │  (Docker image)  │   │
│  └────────┬──────────┘   └──────────┬───────────┘   └────────┬─────────┘   │
└───────────│──────────────────────────│────────────────────────│─────────────┘
            │                          │                        │
            └──────────────────────────┼────────────────────────┘
                                       │  HTTPS / REST
                                       │  + SSE (event stream)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY LAYER                                 │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  FastAPI  (Python 3.12)                                              │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐  ┌─────────┐  │   │
│  │  │ /compile     │  │ /runs (CRUD) │  │ /runs/{id}  │  │ /events │  │   │
│  │  │ POST         │  │              │  │ /approve     │  │ SSE     │  │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬──────┘  └────┬────┘  │   │
│  └─────────│─────────────────│─────────────────│──────────────│────────┘   │
│            │                 │                 │              │             │
│        Clerk JWT          Clerk JWT         Clerk JWT     Redis subscribe   │
└────────────│─────────────────│─────────────────│──────────────│─────────────┘
             │                 │                 │              │
     ┌───────┘           ┌─────┘                │              │
     ▼                   ▼                       │              ▼
┌──────────────┐   ┌──────────────────────────────────────────────────────┐
│  COMPILER    │   │               RUNTIME ORCHESTRATOR                   │
│  SERVICE     │   │                                                      │
│  (stateless) │   │  IRWalker (per-run ECS task)                        │
│              │   │  ┌────────────┐  ┌──────────────┐  ┌─────────────┐  │
│  lexer       │   │  │ Dispatch   │  │ Context Mgr  │  │ State       │  │
│  parser      │   │  │ Table      │  │ tiktoken     │  │ Machine     │  │
│  ast_nodes   │   │  │ (op→handler│  │ compression  │  │ (ir_index,  │  │
│  task_graph  │   │  │ dispatch)  │  │ PUSH/POP     │  │ variables,  │  │
│  symbol_table│   │  └────────────┘  └──────────────┘  │ retry_count)│  │
│  type_checker│   │                                     └─────────────┘  │
│  ir_emitter  │   └──────────────────────────────────────────────────────┘
│              │          │                    │
│  IR JSON out │    ┌─────┘             ┌──────┘
└──────────────┘    │                   │
                    ▼                   ▼
          ┌──────────────────┐   ┌─────────────────────────────────────────┐
          │  TOOL EXECUTOR   │   │            MODEL ROUTER                 │
          │                  │   │                                         │
          │  fs.glob         │   │  alias → model_name → tier selection   │
          │  fs.read/write   │   │                                         │
          │  shell.run       │   │  ┌──────────┐  ┌──────────┐  ┌───────┐ │
          │   (Docker        │   │  │  TIER 1  │  │  TIER 2  │  │TIER 3 │ │
          │    sandbox)      │   │  │  Bedrock │  │  vLLM    │  │ vLLM  │ │
          │  human.checkpoint│   │  │  Claude  │  │  26B-A4B │  │  12B  │ │
          │  retrieve        │   │  │  Llama 3 │  │  ~60t/s  │  │ ~80t/s│ │
          │  http.*          │   │  │  Mistral │  │  A10G    │  │  g5xl │ │
          └──────────────────┘   │  └──────────┘  └──────────┘  └───────┘ │
                  │              │      │               │              │    │
                  │              └──────│───────────────│──────────────│────┘
                  │                     │               │              │
                  ▼                     ▼               ▼              ▼
          ┌──────────────┐    ┌──────────────┐  ┌──────────────────────────┐
          │  S3 ARTIFACT │    │ AWS BEDROCK  │  │   GPU SERVING LAYER      │
          │  STORAGE     │    │ (HTTPS/IAM)  │  │   vLLM + AWQ + MTP       │
          │  per-tenant  │    │              │  │   3× g5.xlarge spot       │
          │  prefix      │    │              │  │   US-EAST / EU-WEST / APAC│
          └──────────────┘    └──────────────┘  └──────────────────────────┘
                                     │
                              ┌──────┴──────┐
                              │  OpenAI     │
                              │  (optional  │
                              │   fallback) │
                              └─────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         PERSISTENCE + QUEUE LAYER                           │
│                                                                             │
│  ┌──────────────────────┐    ┌─────────────────────┐   ┌─────────────────┐ │
│  │  RDS POSTGRES        │    │  ELASTICACHE REDIS  │   │  STRIPE BILLING │ │
│  │  Multi-AZ            │    │                     │   │                 │ │
│  │  Row-Level Security  │    │  pub/sub: SSE events│   │  Subscriptions  │ │
│  │  tenants             │    │  per run_id channel │   │  Metered usage  │ │
│  │  pipelines           │    │                     │   │  Credit packs   │ │
│  │  pipeline_runs       │    │  RQ: async run      │   │  Webhooks       │ │
│  │  run_events          │    │  dispatch queue     │   │                 │ │
│  │  model_calls         │    │                     │   │                 │ │
│  │  tool_calls          │    │                     │   │                 │ │
│  │  billing_usage       │    │                     │   │                 │ │
│  └──────────────────────┘    └─────────────────────┘   └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Flow: Compile + Run

The following sequence traces exactly what happens from the moment a user clicks **Run** in the Web IDE to the moment the dashboard shows "completed."

**Step 1 — Compile request**
The Web IDE sends `POST /compile` with the `.agent` source body. The FastAPI gateway validates the Clerk JWT, extracts `tenant_id`, and forwards the source to the Compiler Service (same process, separate module). The compiler runs the full pipeline: lex → parse → AST → task graph → symbol table → type check → IR emit. On success, it returns `{ "ir": [...], "warnings": [...] }`. On failure, it returns `{ "errors": [...] }` with line/column annotations. Total compiler latency target: <100ms p50.

**Step 2 — Run creation**
If compilation succeeds, the Web IDE sends `POST /runs` with `{ pipeline_id, ir, config }`. The gateway creates a `pipeline_runs` row with `status='pending'` and the full `ir_json`. It enqueues a run dispatch job in Redis Queue (RQ) with the `run_id`. It returns `{ run_id, stream_url }` to the client.

**Step 3 — SSE subscription**
The Web IDE immediately connects to `GET /runs/{run_id}/events` — a long-lived SSE connection. The FastAPI endpoint subscribes to the Redis pub/sub channel `run:{run_id}:events`. Every event the runtime publishes appears here in real time.

**Step 4 — RQ worker dispatch**
An RQ worker picks up the dispatch job. It starts a new ECS Fargate task for the `runtime-orchestrator` image, passing `run_id` as an environment variable. (For low-traffic environments, the worker can execute the IRWalker in-process instead of spawning a task.)

**Step 5 — Runtime boot**
The orchestrator task starts, loads the `pipeline_runs` row from Postgres by `run_id`, restores `variables_json` and `context_stack_json`, and sets `status='running'`. It publishes a `run_started` event to Redis.

**Step 6 — IR walk**
For each instruction in `ir[ir_index:]`:
1. Dispatch to the appropriate handler (LLM_CALL, CALL, BRANCH, etc.)
2. On LLM_CALL: the model router selects the tier, calls the provider, receives a response, parses content + token counts, posts a Stripe usage record, inserts a `model_calls` row, assigns the output variable.
3. On CALL: the tool executor runs the handler (e.g., spawns a Docker container for `shell.run`), captures output, assigns the output variable.
4. On CHECKPOINT: serialize display data, publish a `checkpoint` SSE event, raise `CheckpointException`, set `status='paused'`, persist state, return.
5. After each instruction: increment `ir_index`, persist `pipeline_runs` row, publish a `step_completed` event.

**Step 7 — Human checkpoint resume**
The Web IDE renders the checkpoint data (e.g., a list of dead code candidates). The user clicks Approve. The IDE sends `POST /runs/{run_id}/approve` with `{ decision: "approved" }`. The gateway writes the decision into the run's `variables_json` under the CHECKPOINT output variable, sets `status='running'`, re-enqueues the run. The IRWalker resumes at `ir_index + 1`.

**Step 8 — Completion**
When the walker reaches HALT or the last instruction, it sets `status='completed'`, writes `completed_at`, publishes `run_completed` event with a summary. The Web IDE SSE stream receives the event and updates the dashboard.

**Step 9 — Billing finalization**
The runtime posts a `billing_usage` row after each LLM_CALL. A nightly job aggregates usage by tenant and period and posts Stripe usage records for any gaps. The monthly Bedrock reconciliation job (Lambda, scheduled via CloudWatch Events) queries CloudWatch Bedrock metrics per tenant tag, calculates actual spend, and creates Stripe invoice line items.

---

## 4. External Integrations

| Integration | Purpose | Protocol | Auth | SLA Dependency |
|---|---|---|---|---|
| AWS Bedrock | Tier 1 LLM inference | HTTPS | IAM role (ECS task role) | **Hard** — no fallback within Tier 1; Bedrock outage = Tier 1 unavailable |
| OpenAI | Tier 1 optional / overflow | HTTPS | API key (KMS-encrypted per tenant) | Soft — fall back to Bedrock Claude |
| Anthropic API | Tier 1 direct Claude access | HTTPS | API key (KMS-encrypted per tenant) | Soft — fall back to Bedrock Claude |
| vLLM (private GPU) | Tier 2/3 LLM inference | HTTPS (OpenAI-compatible) | Bearer token (internal) | Soft — fall back to Bedrock on GPU outage |
| Stripe | Subscription billing + usage metering | HTTPS REST + Webhooks | Webhook signing secret | Soft — billing paused, execution continues |
| Clerk | JWT issuance + user management | HTTPS | JWT RS256 verification | **Hard** — no auth = no API access |
| AWS S3 | Artifact storage (fs.write outputs) | AWS SDK (HTTPS) | IAM role | Soft — artifact writes fail gracefully; run continues |
| AWS KMS | Encryption of per-tenant API keys | AWS SDK | IAM role | **Hard** — cannot decrypt model keys without KMS |
| AWS CloudWatch | Bedrock token usage metrics for reconciliation | AWS SDK | IAM role | Soft — affects monthly billing accuracy only |
| GitHub | CI/CD webhooks, conductor-action | HTTPS Webhook | GitHub App private key | Optional |
| MCP servers | External tool integrations | HTTP/SSE | OAuth2 | Optional |

---

## 5. Scalability Design

**Compiler Service**
Stateless Python/FastAPI process deployed on ECS Fargate. Each request is fully self-contained: source bytes in, IR JSON out. No shared memory, no database writes. Target: <200ms p50. Scale trigger: CPU utilization >60% for 2 consecutive minutes → ECS service auto-scales +2 tasks. Maximum tasks: 20 (prevents cost runaway).

**Runtime Orchestrator**
Stateful per-run. Designed as ECS task-per-run for strong isolation (one pipeline's infinite loop cannot affect others). State persists to Postgres at every IR step — if the task is killed (spot interruption, OOM), the run resumes from the last committed `ir_index`. For low-volume tenants (<10 concurrent runs), runs can execute in the RQ worker process directly to save ECS task startup overhead (~5s).

**Redis Queue**
RQ workers (2–8 instances on ECS) consume the dispatch queue. Each worker picks up a run dispatch job and either executes in-process (small runs) or starts an ECS task (large/long runs). Queue depth alarm: if queue depth >50 for 5 minutes, scale RQ workers.

**GPU Tier**
Manual scaling for early stage: add g5.xlarge spot instances as MAU grows. Each instance serves ~60 tok/s at full utilization. Autoscale trigger (when active user count justifies automation): CloudWatch custom metric `gpu_queue_depth` → Lambda function → update ASG desired count. Spot interruption handled by checkpoint/resume (see gpu-serving-design.md).

**Postgres**
RDS PostgreSQL 15, Multi-AZ. Read replicas for dashboard queries (billing aggregates, run history). Connection pooling via PgBouncer sidecar on each ECS service. Target: <10ms p50 on indexed queries.

**Context Budget Enforcement**
The Context Manager (per-run component) counts tokens with tiktoken before each LLM_CALL. If the assembled context exceeds the pipeline's declared `context budget`, it runs a SUMMARIZE pass (cheapest available model) on the oldest context stack entry. This prevents a runaway context from consuming the account's credit balance without bound.

---

## 6. Security Design

**Authentication and Authorization**
All API endpoints require a Clerk JWT (`Authorization: Bearer <token>`). The JWT is verified against Clerk's JWKS endpoint (RS256). The `tenant_id` is extracted from the JWT `org_id` claim. Every database query runs with `SET LOCAL app.tenant_id = '<tenant_id>'` so RLS policies apply for the request lifetime.

**Secret Management**
Per-tenant API keys (Bedrock, OpenAI, Anthropic) are encrypted with AWS KMS before storage in the `tenants.api_key_encrypted` column. Each tenant has a dedicated KMS key (`tenants.kms_key_id`). The ECS task role has `kms:Decrypt` permission scoped to the tenant's key prefix. Stripe API keys and webhook secrets are stored in AWS Secrets Manager and injected as environment variables at ECS task launch.

**Shell Sandbox**
`shell.run` executes user-specified commands inside a Docker container with:
- CPU limit: 1 core
- Memory limit: 512MB
- Network: blocked (no outbound, no inbound)
- Filesystem: ephemeral overlay; host filesystem not mounted
- Seccomp profile: Docker default (blocks 44 dangerous syscalls)
- Execution timeout: 120s hard kill

**Prompt Injection Defense**
Retrieved content (from `retrieve()` calls or `fs.read`) is sanitized before injection into LLM prompts:
- Strip XML-like tags that could confuse system/user message boundaries
- Prepend a delimiter: `--- BEGIN RETRIEVED CONTENT ---` / `--- END RETRIEVED CONTENT ---`
- Log retrieved content hash in `run_events` for auditability

**Rate Limiting**
Redis token bucket per tenant: 100 API requests/minute for free tier, 1000/minute for Builder+. Enforced at FastAPI middleware layer before the request reaches any handler. Exceeded tenants receive HTTP 429 with `Retry-After` header.

**TLS**
All inter-service communication uses TLS 1.3. Internal ECS services communicate via AWS VPC (private subnets), but still require TLS for defense-in-depth. GPU serving endpoints require a bearer token even on internal network.

**JWT Expiry**
Access tokens: 1-hour expiry. Refresh tokens: 30-day expiry, rotated on use. The Web IDE handles silent refresh via Clerk's SDK.

---

## 7. Disaster Recovery

**Postgres**
RDS Multi-AZ deployment: automated failover to standby replica on primary failure (typically <60s). Point-in-time recovery enabled with 7-day backup retention. Automated daily snapshots retained for 30 days. In a region-level outage: restore from snapshot to a new RDS instance in a backup region. RTO: ~2 hours for region failover. RPO: up to 5 minutes (replication lag at failure moment).

**Redis**
ElastiCache Redis with cluster mode enabled (3 shards, 1 replica each). On primary shard failure, replica is promoted automatically. SSE pub/sub events in-flight at failure time are lost (clients must reconnect and re-subscribe; run state is in Postgres, not Redis). RQ job data is persisted to Redis AOF; jobs survive a Redis restart.

**GPU Serving**
Spot instance interruption is handled at the application level (see gpu-serving-design.md). Because the runtime checkpoints to Postgres after every IR instruction, a GPU-backed LLM call that is in-flight at interruption time is retried on the next available instance (same or different region). The run does not lose earlier completed steps.

**ECS Tasks (Compiler + Runtime)**
ECS Fargate tasks are replaced automatically on failure by the ECS service scheduler. Compiler tasks are stateless — replacement is immediate. Runtime tasks are per-run and stateful, but state is in Postgres; the replacement task resumes from `ir_index` on start.

**RTO Target:** 4 hours (full service restoration after region failure).
**RPO Target:** 1 hour (maximum data loss window).

**Runbook Locations**
- `ops/runbooks/postgres-failover.md` — step-by-step RDS failover and DNS cutover
- `ops/runbooks/redis-restore.md` — ElastiCache cluster recovery
- `ops/runbooks/gpu-spot-recovery.md` — manual GPU instance replacement
- `ops/runbooks/stripe-outage.md` — billing pause policy during Stripe degradation
