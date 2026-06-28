---
type: PRD Phase
title: "Phase 2 — Growth: Ecosystem Integration + Team Features"
description: Phase 2 PRD for Conductor covering VS Code extension, conductor CLI, GitHub Actions, MCP server and client integration, pipeline template marketplace, metrics dashboard, team workspaces, OpenAI provider support, and LangChain migration tooling. Exit gate is 100 monthly active users.
resource: /startup/02-prd/overview.md
tags: [prd, phase-2, growth, vscode, cli, github-actions, mcp, templates, team, openai, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Phase 2 — Growth: Ecosystem Integration + Team Features

## Phase Goal

Conductor leaves the confines of the web IDE and enters the developer's daily workflow. By the end of Phase 2, a developer can write a `.agent` file in VS Code with syntax highlighting and error squiggles, run it from the terminal with a single `conductor run` command, trigger it automatically in CI via a GitHub Actions step, and reference any MCP-compatible tool server from within a pipeline. Teams can share pipelines, track per-pipeline metrics, and bill to a single team Stripe subscription. Five production-ready templates lower the barrier to first meaningful pipeline.

## Phase Exit Gate

- 100 monthly active users (at least one pipeline run per user per month)
- VS Code extension published to the VS Code Marketplace with 50+ installs
- MCP compliance test suite passing (all protocol-level tests)
- At least 2 of the 5 templates used by more than 10 distinct users
- At least 3 teams on the Team tier ($99/seat/mo)

## Architecture Decisions Locked in Phase 2

- **CLI**: Python Click-based; distributed as a standalone binary via PyInstaller for Linux/macOS/Windows
- **VS Code extension**: TypeScript, VSCode Extension API; TextMate grammar for syntax highlighting, Language Server Protocol for error squiggles
- **GitHub Actions**: Docker-based action (`conductorlabs/conductor-action`); Docker image published to GHCR
- **MCP**: Conductor implements MCP server spec v0.1; tools auto-generated from `conductor/tool_registry.py` type signatures
- **OpenAI provider**: `openai` Python SDK; key stored encrypted (AES-256-GCM) in `tenants.provider_config JSONB`

---

## Tasks

### TASK-201: VS Code Extension
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-106 (IR Emitter — needed for compile invocation), TASK-202 (CLI — extension shells out to CLI)
**Acceptance criteria**:
- [ ] Extension scaffolded at `vscode-extension/` using `yo code` (TypeScript template)
- [ ] TextMate grammar in `vscode-extension/syntaxes/agent.tmLanguage.json` with scopes for: pipeline/step/model/prompt/output/branch/condition/loop/on/tool (keywords), `$variable` (variable.other), string literals (string.quoted), comments (comment.line.number-sign)
- [ ] `.agent` file association registered: `"languages":[{"id":"agent","extensions":[".agent"]}]`
- [ ] Hover provider: hovering over a keyword shows a one-sentence description of its role in the pipeline DSL; hovering over a built-in tool name (`fs.glob`, `shell.run`, etc.) shows its signature and return type from `conductor/tool_registry.py`
- [ ] "Conductor: Compile" command (`conductor.compile`) in command palette: shells out to `conductor compile <file>`, captures JSON output, populates VS Code `DiagnosticCollection` with errors/warnings at correct line/col
- [ ] "Conductor: Run" command (`conductor.run`) in command palette: shells out to `conductor run <file>`, streams stdout to a dedicated Output Channel named "Conductor"
- [ ] Error squiggles visible in editor when compile returns type errors
- [ ] Extension packaged as `.vsix` and published to VS Code Marketplace (manual publish step documented in `vscode-extension/PUBLISHING.md`)
- [ ] Works on VS Code 1.85+ on Linux, macOS, and Windows
- [ ] 10-case integration test suite using `@vscode/test-electron`: syntax highlighting scopes asserted, compile command tested against a fixture `.agent` file

**Agent context needed**:
- VS Code Extension API docs (TextMate grammars, DiagnosticCollection, Language Configuration)
- `conductor/tool_registry.py` (TASK-105 output — for hover docs)
- `conductor` CLI `compile` command output format (TASK-202 output — write TASK-202 first)

**Deliverable**: `vscode-extension/`, `vscode-extension/syntaxes/agent.tmLanguage.json`, `vscode-extension/PUBLISHING.md`

---

### TASK-202: Conductor CLI
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-108 (State Machine), TASK-111 (FastAPI Backend — for remote run mode)
**Acceptance criteria**:
- [ ] `conductor` CLI entry point in `conductor/cli.py` using Click; registered as console script in `pyproject.toml`
- [ ] `conductor compile <file>` subcommand: reads `.agent` source, runs lexer → parser → AST → symbol table → type checker → IR emitter, prints IR as JSON to stdout (default) or errors as human-readable text with `--format text`; exits 0 on success, 1 on compile error
- [ ] `conductor run <file>` subcommand: compiles then dispatches against configured provider; streams execution events to stdout as line-delimited JSON (`{"event": "step_completed", "step": "FindDeadCode", "tokens": 1240}`); exits 0 on success, 2 on runtime error, 3 on budget exceeded
- [ ] `conductor run --dry-run <file>`: compiles and type-checks, logs each IR instruction with `[DRY RUN]` prefix, makes no API calls; exits 0 if valid, 1 if compile error
- [ ] `conductor run --remote <file>`: submits run to Conductor cloud API (configured API key in `conductor.yaml`); streams SSE events to stdout; enables use of cloud GPU tiers
- [ ] Config file: `conductor.yaml` in project root or `~/.conductor/config.yaml`; keys: `default_model`, `bedrock.region`, `bedrock.access_key_id`, `bedrock.secret_access_key`, `openai.api_key`, `budget.max_tokens`, `tier` (free/builder/team)
- [ ] `conductor init` subcommand: creates `conductor.yaml` interactively (prompts for provider, API key)
- [ ] `conductor version` subcommand: prints version from `pyproject.toml`
- [ ] Binary distributions built with PyInstaller targeting Linux x86_64, macOS arm64, macOS x86_64 — GitHub Actions release workflow produces binaries on tag push
- [ ] 20-case test suite: compile success/error paths, dry-run, config file parsing, help text

**Agent context needed**:
- `conductor/runtime/state_machine.py` (TASK-108 output)
- Click docs (subcommands, option parsing, streaming output)
- PyInstaller docs (one-file binary, hidden imports)

**Deliverable**: `conductor/cli.py`, `conductor.yaml.example`, `pyproject.toml` (updated with CLI entry point), `.github/workflows/release.yml`

---

### TASK-203: GitHub Actions Integration
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-202 (CLI — action uses CLI binary)
**Acceptance criteria**:
- [ ] Docker-based GitHub Action defined in `github-action/action.yml` with inputs: `pipeline` (required, path to `.agent` file), `api_key` (optional, Conductor cloud API key for remote runs), `dry_run` (optional, default false), `working_directory` (optional, default `.`)
- [ ] Action Docker image based on `python:3.12-slim`; `conductor` CLI installed from release binary
- [ ] Action outputs: `run_id` (Conductor run ID), `status` (completed/failed), `total_tokens`, `total_cost_usd`
- [ ] When `dry_run: true`, action runs `conductor run --dry-run` and succeeds/fails based on compile result only
- [ ] Step summary written to `$GITHUB_STEP_SUMMARY`: markdown table of pipeline steps, status, tokens, cost; link to Conductor web dashboard for the run
- [ ] GitHub Secrets injected via standard `env:` map; docs show pattern for injecting `CONDUCTOR_API_KEY` and `AWS_ACCESS_KEY_ID`
- [ ] Docker image published to `ghcr.io/conductorlabs/conductor-action` on every release tag
- [ ] README at `github-action/README.md` with usage example (copy-paste ready YAML block)
- [ ] Example workflow at `github-action/examples/on-pr-review.yml`: runs `code-review-pr.agent` pipeline on every opened PR, posts results to PR as a comment via `gh pr comment`
- [ ] Integration test: Docker build completes; action runs against `hello-world.agent` in test mode with mocked Bedrock

**Agent context needed**:
- GitHub Actions Docker action docs
- `conductor` CLI output format (TASK-202 output)
- `GITHUB_STEP_SUMMARY` markdown docs

**Deliverable**: `github-action/action.yml`, `github-action/Dockerfile`, `github-action/README.md`, `github-action/examples/`

---

### TASK-204: MCP Protocol Server
**Owner**: Coding Agent
**Complexity**: L
**Depends on**: TASK-111 (FastAPI Backend), TASK-105 (Tool Registry)
**Acceptance criteria**:
- [ ] MCP server implemented in `conductor/mcp/server.py` using `mcp` Python SDK (Anthropic's MCP SDK)
- [ ] Server exposes all tools in `conductor/tool_registry.py` as MCP tools: `fs_glob`, `fs_read`, `fs_write`, `shell_run`, `human_checkpoint`, `retrieve`, `summary`
- [ ] MCP tool definitions auto-generated from `ToolSignature` objects in `tool_registry.py`: name, description, input schema (JSON Schema), output schema inferred from Python type annotations
- [ ] MCP server accessible via: (a) stdio transport (`conductor mcp stdio`) for local clients like Claude Desktop; (b) HTTP/SSE transport on `POST /mcp` FastAPI route for remote clients
- [ ] Tool invocations authenticated via Conductor API key (passed in MCP client config `Authorization: Bearer <key>`)
- [ ] `fs.write` and `shell.run` tools gated behind explicit confirmation in MCP context (MCP `requiresConfirmation: true` flag)
- [ ] MCP server metadata: `{"name": "conductor", "version": "1.0.0", "description": "Conductor tool runtime — filesystem, shell, and LLM pipeline tools"}`
- [ ] Claude Desktop integration documented in `docs/mcp-claude-desktop.md`: step-by-step config for `claude_desktop_config.json`
- [ ] All MCP protocol compliance tests passing (use MCP Inspector tool or equivalent)
- [ ] 15-case test suite: tool listing, tool invocation (mocked), authentication rejection

**Agent context needed**:
- MCP Python SDK docs (`mcp` package from Anthropic)
- `conductor/tool_registry.py` (TASK-105 output)
- MCP protocol spec v0.1 (tool definitions, transport types)

**Deliverable**: `conductor/mcp/server.py`, `conductor/mcp/tool_definitions.py`, `docs/mcp-claude-desktop.md`, updates to `conductor/api/main.py`

---

### TASK-205: MCP Client Integration in Runtime
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-204 (MCP Server), TASK-107 (Bedrock Dispatcher)
**Acceptance criteria**:
- [ ] `.agent` DSL extended with external tool source declaration (lexer + parser + AST builder updated):
  ```agent
  tool github_mcp:
    protocol: mcp
    server: "https://github-mcp.example.com/mcp"
    auth: env(GITHUB_TOKEN)
  ```
- [ ] Lexer recognizes new keywords: `protocol`, `server`, `auth`, `env`
- [ ] Parser emits `ExternalToolSourceNode(name, protocol, server_url, auth_config)`
- [ ] Symbol table tracks external tool sources alongside built-in tools
- [ ] Type checker: external MCP tools fetched at compile time (`conductor compile` calls the MCP server's `tools/list`); their JSON Schema inputs/outputs mapped to Conductor types for type checking
- [ ] IR emitter: `CALL` instruction for an external MCP tool includes `source: "mcp"` and `server_url` fields
- [ ] Bedrock Dispatcher: `CALL` handler for `source: "mcp"` sends `tools/call` request to MCP server with auth header; maps response to Conductor `Any` type
- [ ] Timeout on MCP calls: 30s default, configurable in `conductor.yaml` as `mcp.timeout_seconds`
- [ ] 10-case test suite: external tool declaration parsed correctly, type checking with mocked MCP server tool list, dispatch to mocked MCP server, auth header verified

**Agent context needed**:
- `conductor/lexer.py`, `conductor/parser.py`, `conductor/ast_builder.py` (TASK-101–103 outputs)
- `conductor/runtime/bedrock.py` (TASK-107 output)
- MCP `tools/call` and `tools/list` protocol spec

**Deliverable**: Updated `conductor/lexer.py`, `conductor/parser.py`, `conductor/ast_builder.py`, `conductor/ir.py`; updated `conductor/runtime/bedrock.py`

---

### TASK-206: Pipeline Template Marketplace v1
**Owner**: Both (Coding Agent builds the infrastructure; Founder writes/curates the 5 templates)
**Complexity**: M
**Depends on**: TASK-112 (Web IDE Editor), TASK-113 (Run Dashboard)
**Acceptance criteria**:
- [ ] 5 template `.agent` files committed to `templates/` directory with subdirectory per template:
  1. `templates/refactor-dead-code/`: pipeline.agent + README.md + example output
  2. `templates/summarize-codebase/`: pipeline.agent + README.md + example output
  3. `templates/generate-tests/`: pipeline.agent + README.md + example output
  4. `templates/code-review-pr/`: pipeline.agent + README.md + example output (uses GitHub MCP tool)
  5. `templates/migrate-api-version/`: pipeline.agent + README.md + example output
- [ ] Each template README documents: what the pipeline does, required env vars, expected inputs, sample output
- [ ] Templates API: `GET /templates` returns list of templates with name, description, tags; `GET /templates/{slug}` returns template source
- [ ] Web IDE: "Templates" tab lists all 5 templates with name, description, and "Open in Editor" button; clicking opens the template source in the Monaco editor (replacing current content, with a confirmation dialog)
- [ ] Community submission process: `CONTRIBUTING.md` at repo root documents how to submit a template via GitHub PR; PR template includes checklist (README, example output, type-checking passes)
- [ ] Template usage tracked: when a user opens a template in the IDE, `analytics_events` table records `{event: "template_opened", template_slug, user_id, tenant_id}` — no PII beyond user_id
- [ ] All 5 templates compile without errors (`conductor compile` exits 0 on each)

**Agent context needed**:
- `web/src/` (TASK-112 output)
- `conductor/api/main.py` (TASK-111 output — add `/templates` routes)
- The `.agent` DSL example from product context as the basis for `refactor-dead-code`

**Deliverable**: `templates/`, updates to `conductor/api/`, updates to `web/src/`, `CONTRIBUTING.md`

---

### TASK-207: Metrics Dashboard
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-113 (Run Dashboard), TASK-109 (Postgres Schema)
**Acceptance criteria**:
- [ ] "Metrics" page added to web IDE navigation
- [ ] Per-pipeline metrics panel: shows for each pipeline over last 30 days: total runs, success rate (%), avg tokens per run, avg cost per run (USD), avg duration (seconds); data from new `GET /metrics/pipelines` endpoint
- [ ] Per-step drill-down: clicking a pipeline row expands to show per-step breakdown: step name, avg LLM tokens, avg tool calls, avg duration, failure count
- [ ] Account-level summary: total spend this month (USD), executions used vs. limit (progress bar), top 3 pipelines by cost, top 3 pipelines by run count
- [ ] Export: "Download CSV" button on pipeline metrics table; CSV has columns: pipeline_name, date, runs, successes, failures, avg_tokens, avg_cost_usd
- [ ] Backend: `GET /metrics/pipelines` and `GET /metrics/account` endpoints added to FastAPI; queries aggregated from `model_calls`, `pipeline_runs`, `run_steps` tables; tenant-isolated
- [ ] Metrics queries use Postgres materialized views refreshed every 15 minutes to avoid slow ad-hoc aggregation on large `model_calls` tables
- [ ] Dashboard loads in <2s for accounts with up to 10k runs (tested with seeded data)

**Agent context needed**:
- `conductor/api/main.py` (TASK-111 output)
- `conductor/db/schema.sql` (TASK-109 output)
- `web/src/` (TASK-112/113 output)
- Recharts or Nivo for React charting

**Deliverable**: `web/src/MetricsDashboard.tsx`, `conductor/api/routes/metrics.py`, new Alembic migration for materialized views

---

### TASK-208: Team Workspaces
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-114 (Auth + Tenant Setup), TASK-110 (Stripe Billing)
**Acceptance criteria**:
- [ ] Tenant model extended: `tenants.plan` can be `free`, `builder`, or `team`; `tenants.seat_count INT` added via Alembic migration
- [ ] `GET /team/members` returns list of team members with: user_id, email, role (admin/developer/viewer), joined_at, executions_this_month
- [ ] `POST /team/invite` sends email invitation (via Resend API) with a sign-up link pre-associated with the tenant; `team_invitations(id, tenant_id, email, role, expires_at, accepted_at)` table tracks pending invites
- [ ] Accepted invitations create user row with correct tenant_id on Clerk `user.created` webhook
- [ ] Roles enforced: `viewer` cannot create or run pipelines (HTTP 403 on `POST /runs`); `developer` can create/run; `admin` can invite/remove members and manage billing
- [ ] Shared pipeline library: pipelines with `visibility: team` are returned in `GET /pipelines` for all team members; `POST /pipelines/{id}/fork` creates a copy in the requester's namespace
- [ ] Per-member execution tracking: `billing_usage` table extended with `user_id` column; team dashboard shows breakdown by member
- [ ] Stripe: Team tier uses per-seat pricing; `POST /team/invite` triggers Stripe subscription quantity update (+1 seat); removing member triggers quantity update (-1 seat)
- [ ] 15-case test suite: invite flow, role enforcement (viewer blocked), team pipeline visibility, seat billing

**Agent context needed**:
- `conductor/db/schema.sql` (TASK-109 output)
- `conductor/billing.py` (TASK-110 output)
- Resend API docs (transactional email)

**Deliverable**: `conductor/api/routes/team.py`, updated `conductor/db/`, updated `conductor/billing.py`, `web/src/TeamSettings.tsx`

---

### TASK-209: OpenAI Provider Integration
**Owner**: Coding Agent
**Complexity**: S
**Depends on**: TASK-107 (Bedrock Dispatcher), TASK-111 (FastAPI Backend)
**Acceptance criteria**:
- [ ] `OpenAIDispatcher` in `conductor/runtime/openai_provider.py` implementing the same `dispatch(instruction, state) -> state` interface as `BedrockDispatcher`
- [ ] `LLM_CALL` handler uses `openai.AsyncOpenAI` client; models `gpt-4o` and `gpt-4o-mini` supported
- [ ] `.agent` files can declare `model codegen: gpt-4o` or `model fast: gpt-4o-mini`; compiler resolves these to `OpenAIDispatcher` at runtime
- [ ] Model routing table in `conductor/runtime/router.py`: maps model alias → dispatcher class based on model prefix (`gpt-*` → OpenAI, `claude-*` → Bedrock, `gemma4-*` → vLLM)
- [ ] OpenAI API key stored encrypted in `tenants.provider_config JSONB` field: `{"openai": {"api_key_enc": "<encrypted>"}}`; encryption using `cryptography.fernet` with per-tenant key derived from master secret
- [ ] `POST /settings/providers/openai` endpoint: accepts `{api_key: str}`, encrypts, stores; returns `{configured: true}`
- [ ] Token counting for OpenAI uses `tiktoken` with model-appropriate encoding (`cl100k_base` for GPT-4o)
- [ ] Usage records posted to `model_calls` table as with Bedrock; cost computed using OpenAI published pricing (stored in `conductor/runtime/pricing.py`)
- [ ] 10-case test suite with mocked OpenAI SDK; 1 live integration test gated by `TEST_OPENAI=true`

**Agent context needed**:
- `conductor/runtime/bedrock.py` (TASK-107 output — mirror the interface)
- OpenAI Python SDK docs (`AsyncOpenAI`, `chat.completions.create`)
- `cryptography` Fernet docs

**Deliverable**: `conductor/runtime/openai_provider.py`, `conductor/runtime/router.py`, `conductor/runtime/pricing.py`, `conductor/api/routes/settings.py`

---

### TASK-210: Migration Tool: LangChain/LangGraph → .agent
**Owner**: Coding Agent
**Complexity**: L
**Depends on**: TASK-106 (IR Emitter — for output validation), TASK-202 (CLI — migration is a CLI subcommand)
**Acceptance criteria**:
- [ ] `conductor migrate langchain <input.py> [--output output.agent]` CLI subcommand implemented in `conductor/migrate/langchain.py`
- [ ] AST-based Python parser (using Python `ast` module) identifies patterns: sequential chain (`chain1 | chain2`), LLMChain with prompt template, AgentExecutor with tools, conditional edges in LangGraph `StateGraph`
- [ ] Conversion rules:
  - LangChain `LLMChain(prompt=..., llm=ChatOpenAI(model="gpt-4o"))` → `.agent` step with `model: gpt-4o` and `prompt:` block
  - Sequential chain → ordered steps with data flow edges
  - `AgentExecutor` tool list → `tool:` declarations in corresponding step
  - LangGraph `add_conditional_edges(source, condition_fn, {True: nodeA, False: nodeB})` → `branch:` block
- [ ] For patterns that cannot be auto-converted, migration tool emits a `# TODO: manual conversion required` comment in the output `.agent` file and a line in the migration report
- [ ] Migration report printed to stdout: table of converted constructs, list of unconverted patterns, estimated conversion completeness percentage
- [ ] Output `.agent` file passes `conductor compile` (type-checks) if 100% auto-converted; for partial conversions, compile produces errors only on the `# TODO` stubs
- [ ] 10-case test suite: 6 Python files with known-convertible patterns (assert `.agent` output matches expected fixture), 4 files with partial-conversion patterns (assert report lists correct unconverted items)

**Agent context needed**:
- Python `ast` module docs
- LangChain Expression Language (LCEL) chain composition docs
- LangGraph `StateGraph` docs
- `conductor/cli.py` (TASK-202 output — for subcommand registration)

**Deliverable**: `conductor/migrate/langchain.py`, `tests/fixtures/migration/`, `tests/test_migration.py`

---

## Risk Table

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| VS Code Marketplace review process takes 2+ weeks, delaying extension launch | Medium | Medium | Submit to Marketplace early in Phase 2 with minimal feature set; iterate after initial approval rather than waiting for full polish |
| MCP protocol spec changes between Phase 1 and Phase 2 implementation (spec is still evolving) | High | Medium | Pin to a specific MCP SDK version; abstract MCP calls behind `MCPClient` interface so the protocol adapter is easily swapped; track MCP changelog weekly |
| GitHub Actions Docker image cold start time >30s makes CI integration feel slow | Medium | Medium | Cache dependencies in Docker layer; use BuildKit layer caching in GHCR; target <10s cold start |
| LangChain migration tool produces incorrect `.agent` files that silently change behavior | Medium | High | Scope migration tool to structural conversion only (never infers prompt content); require human review of all migration output; clearly label all auto-converted files with a header comment |
| OpenAI API key storage encryption adds complexity; key rotation not yet supported | Low | High | Document key rotation procedure manually; add key rotation endpoint as TASK-209 follow-up; use Fernet (reversible) not one-way hash so rotation is possible |

---

## Phase Exit Gate Verification

Before declaring Phase 2 complete, verify:

- [ ] VS Code extension: 50 installs on VS Code Marketplace (check Marketplace stats dashboard)
- [ ] 100 MAU: query `SELECT COUNT(DISTINCT user_id) FROM pipeline_runs WHERE started_at > NOW() - INTERVAL '30 days'`
- [ ] MCP compliance: run `npx @modelcontextprotocol/inspector conductor mcp stdio` and confirm all protocol-level tests pass
- [ ] Template usage: query `SELECT template_slug, COUNT(DISTINCT user_id) FROM analytics_events WHERE event='template_opened' GROUP BY template_slug` — at least 2 templates with >10 distinct users
- [ ] Team tier: query `SELECT COUNT(*) FROM tenants WHERE plan='team'` — at least 3
- [ ] GitHub Actions: functional end-to-end test with real GitHub repo (use `conductorlabs/demo` public repo)
