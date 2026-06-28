---
type: PRD Phase
title: "Phase 1 — MVP: Core Harness + Bedrock + Web IDE"
description: Phase 1 PRD for Conductor covering the full compiler pipeline from lexer to IR, the AWS Bedrock runtime dispatcher, execution state machine, Postgres schema, Stripe billing, FastAPI backend, and minimal web IDE. Exit gate is 10 paying Builder-tier users.
resource: /startup/02-prd/overview.md
tags: [prd, phase-1, mvp, compiler, bedrock, web-ide, fastapi, stripe, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Phase 1 — MVP: Core Harness + Bedrock + Web IDE

## Phase Goal

Deliver a working end-to-end Conductor runtime: a developer can write a `.agent` file in a web IDE, click Compile to see the IR and any type errors, click Run to dispatch the pipeline to AWS Bedrock Claude 3.5 Sonnet, and watch step-by-step progress in real time. Human checkpoint steps pause execution and present an approval modal. The full execution log is persisted and resumable after disconnect. All of this is wrapped in a billing system that enforces the free tier (500 IR executions/mo) and charges Builder-tier users ($29/mo) via Stripe.

## Phase Exit Gate

10 users paying $29/mo (Builder tier) actively using Conductor to run real agent pipelines on their own codebases. "Active" means at least one successful pipeline run in the billing period.

## Architecture Decisions Locked in Phase 1

These decisions are made once and not revisited without a formal RFC:

- **Language**: Python 3.12 for compiler + runtime (FastAPI backend); TypeScript + React for web IDE
- **Compiler target**: Linear IR instruction list (not bytecode; see TASK-106)
- **Storage**: Postgres 16 (SQLite for local dev), Redis 7 for run event queue
- **Billing**: Stripe metered billing with per-token usage records
- **Auth**: Clerk (social login: GitHub + Google; no enterprise SSO in Phase 1)
- **Deployment**: Single AWS region (us-east-1); Docker Compose for dev, ECS Fargate for prod
- **Model routing**: AWS Bedrock only in Phase 1 (Claude 3.5 Sonnet, `anthropic.claude-3-5-sonnet-20241022-v2:0`)

---

## Tasks

### TASK-101: Lexer Implementation
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: none
**Acceptance criteria**:
- [ ] `lex(source: str) -> list[Token]` function implemented in `conductor/lexer.py`
- [ ] Token dataclass has fields: `type: TokenType`, `lexeme: str`, `line: int`, `col: int`
- [ ] All token types enumerated: `KEYWORD`, `IDENT`, `STRING`, `NUMBER`, `DOLLAR_IDENT`, `ARROW`, `COLON`, `LBRACKET`, `RBRACKET`, `COMMA`, `LPAREN`, `RPAREN`, `NEWLINE`, `INDENT`, `DEDENT`, `EOF`
- [ ] Keywords recognized: `pipeline`, `step`, `model`, `context`, `tool`, `prompt`, `output`, `branch`, `loop`, `condition`, `on`, `for`, `in`, `default`
- [ ] `$variable_name` lexed as `DOLLAR_IDENT` with lexeme `variable_name` (dollar stripped)
- [ ] `->` lexed as `ARROW`
- [ ] String literals (single and double quoted, multi-line triple-quoted) lexed correctly
- [ ] Indentation-sensitive: INDENT/DEDENT tokens emitted on indentation changes (Python-style)
- [ ] Line comments (`#`) skipped
- [ ] 40-case unit test suite passing (`tests/test_lexer.py`): 30 positive, 10 negative (invalid tokens raise `LexError` with line/col)
- [ ] The canonical `refactor-dead-code.agent` example lexes without errors

**Agent context needed**:
- `wiki/chapters/ch03-lexical-analysis/index.md` and `wiki/chapters/ch03-lexical-analysis/concepts.md`
- The `.agent` DSL example in the product context (pipeline RefactorDeadCode)
- Python `dataclasses` and `enum` stdlib docs (no external dependencies for lexer)

**Deliverable**: `conductor/lexer.py`, `tests/test_lexer.py`, `tests/fixtures/refactor-dead-code.agent`

---

### TASK-102: Parser + Parse Tree
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-101
**Acceptance criteria**:
- [ ] `parse(tokens: list[Token]) -> PipelineNode` implemented in `conductor/parser.py`
- [ ] Recursive descent parser; no parser generator dependency
- [ ] Parse tree node types implemented as dataclasses in `conductor/nodes.py`: `PipelineNode`, `StepNode`, `ModelDeclNode`, `ContextBudgetNode`, `ContextSourceNode`, `ToolCallNode`, `PromptDeclNode`, `OutputDeclNode`, `BranchNode`, `BranchCaseNode`, `LoopNode`, `ConditionNode`, `ConditionCaseNode`
- [ ] `PipelineNode` has: `name: str`, `models: list[ModelDeclNode]`, `context_budget: ContextBudgetNode | None`, `steps: list[StepNode]`
- [ ] `StepNode` has: `name: str`, `model: str | None`, `context: list[ContextSourceNode]`, `tool: ToolCallNode | None`, `prompt: str | None`, `output: str | None`, `branch: BranchNode | None`, `loop: LoopNode | None`, `condition: ConditionNode | None`
- [ ] `ParseError` raised with line/col on malformed input
- [ ] 30-case test suite: 20 positive (full parse trees asserted), 10 negative (expected `ParseError`)
- [ ] The canonical `refactor-dead-code.agent` parses to a valid `PipelineNode` with 5 `StepNode` children

**Agent context needed**:
- `wiki/chapters/ch04-syntax-analysis/index.md`
- `conductor/lexer.py` and `conductor/nodes.py` (produced by TASK-101 + this task)
- CFG description: pipeline → `pipeline` IDENT `:` NEWLINE INDENT (model_decl | budget_decl | step_decl)+ DEDENT

**Deliverable**: `conductor/parser.py`, `conductor/nodes.py`, `tests/test_parser.py`

---

### TASK-103: AST + Task Graph Builder
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-102
**Acceptance criteria**:
- [ ] `build_task_graph(pipeline: PipelineNode) -> TaskGraph` implemented in `conductor/ast_builder.py`
- [ ] `TaskGraph` is a directed graph (using `networkx.DiGraph` or equivalent): nodes are `TaskNode` instances, edges carry a `kind: Literal["data", "control"]` attribute
- [ ] `TaskNode` dataclass has: `id: str`, `step_name: str`, `node_type: Literal["llm_call", "tool_call", "branch", "loop", "checkpoint"]`, `model: str | None`, `context_sources: list[str]`, `prompt: str | None`, `tools: list[str]`, `output_var: str | None`
- [ ] Data edges: drawn from the step that produces `$var` to every step that consumes `$var`
- [ ] Control edges: drawn for `branch` and `condition` outcomes (e.g., `approved -> RefactorEach`)
- [ ] Loop back-edges correctly identified and tagged `kind="loop_back"` — graph is otherwise a DAG
- [ ] `CycleError` raised if non-loop back-edge creates a cycle
- [ ] `graphviz` DOT export: `task_graph.to_dot() -> str` produces valid DOT syntax
- [ ] 15-case test suite including the RefactorDeadCode pipeline (verify node count = 5, edge count ≥ 4)

**Agent context needed**:
- `wiki/chapters/ch05-ast/index.md`
- `conductor/nodes.py` (TASK-102 output)
- networkx documentation for `DiGraph`

**Deliverable**: `conductor/ast_builder.py`, `conductor/task_graph.py`, `tests/test_ast_builder.py`

---

### TASK-104: Symbol Table
**Owner**: Coding Agent
**Complexity**: S
**Depends on**: TASK-103
**Acceptance criteria**:
- [ ] `build_symbol_table(graph: TaskGraph) -> SymbolTable` implemented in `conductor/symbol_table.py`
- [ ] `SymbolTable` maps variable name → `Symbol(name: str, producing_step: str, scope: Literal["pipeline", "loop_local"], type: str | None)`
- [ ] Loop-local variables (`$item` in `loop: for $item in $collection`) are scoped to the loop body and not visible outside
- [ ] `UndefinedVariableError` raised with variable name, referencing step name, line/col when a `$var` is referenced before being produced
- [ ] `RedefinitionError` raised when two steps declare the same `output:` variable name at the same scope
- [ ] Handles forward references correctly: a step can reference `$var` if the `$var`-producing step is guaranteed to precede it in topological order
- [ ] Symbol table entries exported as JSON for use by IR emitter
- [ ] 20-case test suite: 14 positive, 6 negative (undefined ref, redefinition, scope escape)

**Agent context needed**:
- `wiki/chapters/ch06-symbol-tables/index.md`
- `conductor/task_graph.py` (TASK-103 output)

**Deliverable**: `conductor/symbol_table.py`, `tests/test_symbol_table.py`

---

### TASK-105: Type Checker
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-104
**Acceptance criteria**:
- [ ] `type_check(graph: TaskGraph, symbols: SymbolTable) -> list[TypeError]` implemented in `conductor/type_checker.py`
- [ ] Tool signature registry in `conductor/tool_registry.py`:
  - `fs.glob(pattern: str) -> FileList`
  - `fs.read(path: str) -> FileContent`
  - `fs.write(path: str, content: str) -> WriteResult`
  - `shell.run(command: str) -> ExecResult` (fields: `.exit_code: int`, `.stdout: str`, `.stderr: str`)
  - `human.checkpoint(data: Any) -> ApprovalResult` (fields: `.status: Literal["approved","rejected"]`)
  - `retrieve(path: str) -> DocumentChunk` (fields: `.text: str`, `.source: str`)
  - `summary(content: Any) -> String`
- [ ] Type errors raised when: branch/condition field (e.g., `.status`) does not exist on the variable's type; context source type is `ExecResult` passed to a step expecting `FileContent`; `loop: for $item in $var` where `$var` is not iterable (`FileList` is iterable, `String` is not)
- [ ] Each `TypeError` has: `message: str`, `file: str`, `line: int`, `col: int`, `error_code: str` (e.g., `TC001` for unknown field access)
- [ ] Warnings (not errors) emitted for: context sources that may exceed 32k token budget, `retrieve()` paths that don't exist in the repo
- [ ] `type_check()` returns empty list on valid pipelines
- [ ] 25-case test suite: 15 valid pipelines (empty error list), 10 negative (expected specific error codes)

**Agent context needed**:
- `wiki/chapters/ch07-type-system/index.md`
- `conductor/symbol_table.py` (TASK-104 output)
- `conductor/task_graph.py` (TASK-103 output)

**Deliverable**: `conductor/type_checker.py`, `conductor/tool_registry.py`, `tests/test_type_checker.py`

---

### TASK-106: IR Emitter
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-105
**Acceptance criteria**:
- [ ] `emit_ir(graph: TaskGraph, symbols: SymbolTable) -> list[IRInstruction]` implemented in `conductor/ir_emitter.py`
- [ ] IR instruction set as dataclasses in `conductor/ir.py`:
  - `LLM_CALL(model: str, prompt_template: str, context_vars: list[str], output_var: str)`
  - `CALL(tool: str, args: list[Any], output_var: str)`
  - `ASSIGN(var: str, value: Any)`
  - `BRANCH(var: str, field: str, cases: dict[str, int])` (cases map values to IR instruction offsets)
  - `LOOP_START(collection_var: str, item_var: str)`
  - `LOOP_END()`
  - `CONTEXT_PUSH(sources: list[str])`
  - `CONTEXT_POP()`
  - `RETRIEVE(path: str, output_var: str)`
  - `CHECKPOINT(display_var: str, output_var: str)`
  - `JUMP(offset: int)`
  - `HALT()`
- [ ] Task Graph nodes are topologically sorted before linearization
- [ ] Loop bodies are linearized between `LOOP_START` and `LOOP_END` with correct `BRANCH` back-edge to `LOOP_START`
- [ ] IR serializable to/from JSON (for persistence in Postgres and transmission over API)
- [ ] `conductor compile refactor-dead-code.agent --format json` prints valid IR JSON
- [ ] 15-case test suite asserting exact IR instruction sequences for known pipelines

**Agent context needed**:
- `wiki/chapters/ch08-intermediate-rep/index.md`
- `conductor/task_graph.py`, `conductor/symbol_table.py` (prior task outputs)
- Topological sort algorithm (use `networkx.topological_sort`)

**Deliverable**: `conductor/ir.py`, `conductor/ir_emitter.py`, `tests/test_ir_emitter.py`, `conductor/cli.py` (stub with `compile` subcommand)

---

### TASK-107: Bedrock Runtime Dispatcher
**Owner**: Coding Agent
**Complexity**: L
**Depends on**: TASK-106
**Acceptance criteria**:
- [ ] `BedrockDispatcher` class in `conductor/runtime/bedrock.py` with method `dispatch(instruction: IRInstruction, state: RunState) -> RunState`
- [ ] `LLM_CALL` handler: constructs messages array, calls `boto3` `bedrock-runtime` `invoke_model` with `anthropic.claude-3-5-sonnet-20241022-v2:0`, parses response, writes output to `state.vars[output_var]`
- [ ] `CALL fs.glob` handler: uses `pathlib.glob`, returns list of matching paths as `FileList`
- [ ] `CALL fs.read` handler: reads file, returns `FileContent`
- [ ] `CALL fs.write` handler: writes file, returns `WriteResult(success=True, path=...)`
- [ ] `CALL shell.run` handler: `subprocess.run` with timeout (default 60s), captures stdout/stderr/exit_code, returns `ExecResult`
- [ ] `CALL human.checkpoint` handler: persists current state to Postgres with status `PAUSED`, emits SSE event `checkpoint_pending`, returns without blocking (caller polls for resume)
- [ ] `CONTEXT_PUSH` handler: assembles context window from `state.vars`, estimates token count using `tiktoken` (cl100k_base), logs warning if >32k
- [ ] `RETRIEVE` handler: reads file from repo bundle at given path, returns `DocumentChunk`
- [ ] `BRANCH`/`JUMP`/`LOOP_START`/`LOOP_END`/`HALT` handlers update instruction pointer in `RunState`
- [ ] AWS credentials read from environment (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`) — no hardcoded credentials
- [ ] Integration test (live): runs `hello-world.agent` (single LLM_CALL step) against Bedrock, asserts non-empty output (requires `TEST_BEDROCK=true` env flag to run)
- [ ] All handlers have unit tests with mocked boto3 and mocked subprocess

**Agent context needed**:
- `conductor/ir.py` (TASK-106 output)
- AWS Bedrock `invoke_model` API docs (boto3): model ID, request/response format for Claude 3.5 Sonnet
- `harness/README.md` for reliability engineering principles (checkpointing, retry)

**Deliverable**: `conductor/runtime/bedrock.py`, `conductor/runtime/state.py`, `tests/test_bedrock_dispatcher.py`, `tests/fixtures/hello-world.agent`

---

### TASK-108: Execution State Machine + Checkpointing
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-107, TASK-109
**Acceptance criteria**:
- [ ] `RunStateMachine` in `conductor/runtime/state_machine.py` manages the full lifecycle of a pipeline run
- [ ] Run states: `PENDING`, `RUNNING`, `PAUSED`, `COMPLETED`, `FAILED` — stored in `pipeline_runs.status` column
- [ ] After every IR instruction dispatch, state is persisted to Postgres: current instruction pointer, all `state.vars`, step start/end timestamps
- [ ] `RunStateMachine.resume(run_id: str, approval: ApprovalResult | None)` resumes a `PAUSED` run from the persisted checkpoint
- [ ] Idempotency: `CALL fs.write` and `CALL shell.run` instructions carry an `idempotency_key` (hash of run_id + instruction index); if the key exists in `tool_calls` table, the persisted result is returned without re-executing
- [ ] Failed runs store the exception type, message, and stack trace in `run_steps.error_details` (JSON)
- [ ] `RunStateMachine.abort(run_id: str)` sets status to `FAILED` with reason `"user_aborted"`, releases any held resources
- [ ] Integration test: start a run with a `CHECKPOINT` instruction, kill the process, restart, call `resume()`, assert run completes

**Agent context needed**:
- `conductor/runtime/bedrock.py`, `conductor/runtime/state.py` (TASK-107 outputs)
- `conductor/db/schema.sql` (TASK-109 output — write TASK-109 first or stub the schema)
- Postgres `asyncpg` driver docs

**Deliverable**: `conductor/runtime/state_machine.py`, `tests/test_state_machine.py`

---

### TASK-109: Postgres Schema + Migrations
**Owner**: Coding Agent
**Complexity**: S
**Depends on**: none
**Acceptance criteria**:
- [ ] Alembic configured with `conductor/db/alembic.ini` and migration directory `conductor/db/migrations/`
- [ ] Initial migration creates tables:
  - `tenants(id UUID PK, name TEXT, plan TEXT, created_at TIMESTAMPTZ)`
  - `users(id UUID PK, tenant_id UUID FK, email TEXT, role TEXT, created_at TIMESTAMPTZ)`
  - `pipelines(id UUID PK, tenant_id UUID FK, name TEXT, source TEXT, ir_json JSONB, created_at TIMESTAMPTZ, updated_at TIMESTAMPTZ)`
  - `pipeline_runs(id UUID PK, pipeline_id UUID FK, tenant_id UUID FK, status TEXT, started_at TIMESTAMPTZ, completed_at TIMESTAMPTZ, error_details JSONB)`
  - `run_steps(id UUID PK, run_id UUID FK, instruction_index INT, step_name TEXT, status TEXT, started_at TIMESTAMPTZ, completed_at TIMESTAMPTZ, input_vars JSONB, output_vars JSONB, error_details JSONB)`
  - `model_calls(id UUID PK, run_id UUID FK, step_name TEXT, model TEXT, prompt_tokens INT, completion_tokens INT, total_tokens INT, cost_usd NUMERIC(12,6), called_at TIMESTAMPTZ)`
  - `tool_calls(id UUID PK, run_id UUID FK, step_name TEXT, tool TEXT, idempotency_key TEXT UNIQUE, args JSONB, result JSONB, called_at TIMESTAMPTZ)`
  - `billing_usage(id UUID PK, tenant_id UUID FK, period_start DATE, period_end DATE, executions_count INT, bedrock_tokens INT, stripe_usage_record_id TEXT)`
- [ ] Row-level tenant isolation: every query in `conductor/db/queries.py` filters by `tenant_id` from session context
- [ ] `conductor/db/queries.py` provides typed async functions for all CRUD operations needed by TASK-107 and TASK-108
- [ ] `docker-compose.yml` includes Postgres 16 and Redis 7 services
- [ ] `alembic upgrade head` runs cleanly from cold start in CI

**Agent context needed**:
- Alembic docs for async setup with `asyncpg`
- Schema requirements implied by TASK-107 (model_calls, tool_calls) and TASK-110 (billing_usage)

**Deliverable**: `conductor/db/`, `alembic.ini`, `docker-compose.yml`

---

### TASK-110: Stripe Metered Billing Integration
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-109
**Acceptance criteria**:
- [ ] `BillingService` in `conductor/billing.py` with methods: `record_usage(tenant_id, tokens)`, `check_quota(tenant_id) -> bool`, `handle_webhook(payload, sig_header) -> None`
- [ ] 1 credit = 1,000 Bedrock tokens; usage posted to Stripe as metered billing record after each `LLM_CALL` dispatch
- [ ] `check_quota()` returns `False` (and raises `QuotaExceededError`) if free-tier tenant has used 500 executions in current calendar month
- [ ] `handle_webhook()` processes: `customer.subscription.deleted` → downgrade tenant to free tier; `invoice.payment_failed` → send email via Resend API, suspend new runs; `customer.subscription.updated` → update `tenants.plan`
- [ ] Stripe webhook signature verified using `stripe.Webhook.construct_event` before processing
- [ ] Builder tier ($29/mo) Stripe product + price created via Stripe API; IDs stored in environment variables
- [ ] `BillingService` is injected into `RunStateMachine` — quota check runs before `PENDING → RUNNING` transition
- [ ] 15-case test suite with mocked Stripe SDK: usage recording, quota enforcement at limit, webhook event handling

**Agent context needed**:
- Stripe Python SDK docs (metered billing, webhooks)
- `conductor/db/queries.py` (TASK-109 output)
- `conductor/runtime/state_machine.py` (TASK-108 output — for quota injection point)

**Deliverable**: `conductor/billing.py`, `tests/test_billing.py`

---

### TASK-111: FastAPI Backend
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-108, TASK-110
**Acceptance criteria**:
- [ ] FastAPI app in `conductor/api/main.py` with routers in `conductor/api/routes/`
- [ ] `POST /compile` — body: `{source: str}`, response: `{ir: list[IRInstruction], errors: list[TypeError], warnings: list[Warning]}`, HTTP 200 always (errors embedded in body)
- [ ] `POST /runs` — body: `{pipeline_id: str, config: RunConfig}`, response: `{run_id: str}`, starts async run via background task, HTTP 202
- [ ] `GET /runs/{id}` — response: `{run_id, status, steps: list[StepStatus], started_at, completed_at, total_tokens, total_cost_usd}`
- [ ] `GET /runs/{id}/events` — Server-Sent Events stream; events: `step_started`, `step_completed`, `llm_call_started`, `llm_call_completed`, `tool_call_completed`, `checkpoint_pending`, `run_completed`, `run_failed`
- [ ] `POST /runs/{id}/approve` — body: `{decision: "approved"|"rejected"}`, resumes `PAUSED` run, HTTP 200
- [ ] `DELETE /runs/{id}` — aborts run, HTTP 204
- [ ] `GET /health` — returns `{status: "ok", db: "ok", redis: "ok"}`, HTTP 200 (used by ECS health check)
- [ ] JWT auth middleware: validates Clerk JWT on all routes except `/health`; extracts `tenant_id` and `user_id` from JWT claims
- [ ] Request/response models use Pydantic v2
- [ ] OpenAPI schema auto-generated at `/docs`
- [ ] Integration tests with `httpx.AsyncClient` and a real test DB: happy path for compile → run → approve → complete

**Agent context needed**:
- FastAPI docs (SSE with `sse-starlette`, background tasks, Pydantic v2)
- `conductor/runtime/state_machine.py` (TASK-108)
- `conductor/billing.py` (TASK-110)
- Clerk JWT verification docs

**Deliverable**: `conductor/api/`, `tests/test_api.py`

---

### TASK-112: Web IDE — Editor + Compile Panel
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-111
**Acceptance criteria**:
- [ ] React + TypeScript app scaffolded with Vite in `web/`
- [ ] Monaco Editor integrated (`@monaco-editor/react`); editor language set to `agent` (custom language ID)
- [ ] Basic TextMate-style syntax highlighting for `.agent` files: keywords highlighted (pipeline, step, model, prompt, output, branch, condition, loop, on, tool), `$variables` in a distinct color, strings in a distinct color
- [ ] "Compile" button calls `POST /compile` with current editor content
- [ ] On compile success: IR displayed in a read-only panel below the editor (JSON, collapsible per instruction)
- [ ] On compile error: each `TypeError` rendered as a red marker in the Monaco gutter at the correct line/col; errors also listed in a panel below the editor with clickable links that jump to the error line
- [ ] On compile warning: yellow markers in gutter
- [ ] Compile button shows spinner while request in flight; disabled during flight
- [ ] Editor pre-loaded with the canonical `refactor-dead-code.agent` example on first load

**Agent context needed**:
- Monaco Editor React docs
- `conductor/api/main.py` OpenAPI schema (TASK-111 output)
- `tests/fixtures/refactor-dead-code.agent` (TASK-101 output)

**Deliverable**: `web/src/`, `web/package.json`, `web/vite.config.ts`

---

### TASK-113: Web IDE — Run Dashboard
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-112
**Acceptance criteria**:
- [ ] "Run" button in web IDE calls `POST /runs`, then opens the Run Dashboard view
- [ ] Run Dashboard subscribes to `GET /runs/{id}/events` SSE stream
- [ ] Step list renders each step with: name, status badge (pending/running/completed/failed), duration, model called, tokens used
- [ ] Currently-running step has animated "running" state
- [ ] Token budget progress bar: shows tokens used vs. declared context budget; turns red at 90%
- [ ] Cost-per-run display: running total in USD, updated after each `llm_call_completed` event
- [ ] Human checkpoint modal: when `checkpoint_pending` event received, a modal renders the `$display_data` payload (rendered as JSON or markdown), with "Approve" and "Reject" buttons; clicking either calls `POST /runs/{id}/approve`
- [ ] Execution log panel: collapsible per step, shows LLM prompt (truncated to 500 chars with expand), model response (truncated), tool args and result
- [ ] On `run_completed`: green banner with total cost and duration
- [ ] On `run_failed`: red banner with error message and failing step name
- [ ] SSE connection auto-reconnects on disconnect (exponential backoff, max 5 retries)

**Agent context needed**:
- `conductor/api/main.py` SSE event schema (TASK-111 output)
- `web/src/` (TASK-112 output)
- EventSource API docs (browser native SSE)

**Deliverable**: `web/src/RunDashboard.tsx`, `web/src/CheckpointModal.tsx`, and supporting components

---

### TASK-114: Auth + Tenant Setup
**Owner**: Coding Agent
**Complexity**: S
**Depends on**: TASK-111
**Acceptance criteria**:
- [ ] Clerk integration in web IDE: `@clerk/clerk-react` wraps the app; sign-in page shown for unauthenticated users
- [ ] GitHub and Google OAuth configured in Clerk dashboard (documented in `docs/auth-setup.md`)
- [ ] On first sign-in, backend `POST /tenants` called from Clerk webhook (`user.created` event): creates `tenants` row (plan=`free`) and `users` row
- [ ] JWT from Clerk validated in FastAPI middleware using `clerk-backend-api` Python SDK (or JWKS endpoint)
- [ ] Free tier enforcement: `check_quota()` checked before every run start; `QuotaExceededError` returns HTTP 402 with body `{error: "quota_exceeded", executions_used: N, limit: 500}`
- [ ] Web IDE shows current plan and executions remaining in the header (fetched from `GET /account/usage`)
- [ ] `GET /account/usage` endpoint added to FastAPI: returns `{plan, executions_used, executions_limit, bedrock_tokens_used, stripe_customer_portal_url}`
- [ ] Integration test: simulate `user.created` webhook, assert tenant + user rows created

**Agent context needed**:
- Clerk React docs and webhook docs
- `conductor/billing.py` (TASK-110 output)
- `conductor/db/queries.py` (TASK-109 output)

**Deliverable**: `web/src/auth/`, updates to `conductor/api/routes/`, `docs/auth-setup.md`

---

### TASK-115: End-to-End Test Suite
**Owner**: Coding Agent
**Complexity**: M
**Depends on**: TASK-113, TASK-114
**Acceptance criteria**:
- [ ] 3 canonical `.agent` programs compile and run successfully end-to-end (Bedrock calls mocked in CI; live in staging):
  1. `hello-world.agent`: single LLM_CALL step, asserts non-empty string output
  2. `list-and-summarize.agent`: `fs.glob` → `LLM_CALL` → `fs.write`, asserts file written
  3. `refactor-dead-code.agent`: full pipeline with checkpoint, loop, shell.run, asserts exit_code 0
- [ ] Type error test suite: 20 `.agent` files in `tests/fixtures/negative/`, each expected to produce a specific `error_code`; CI asserts all 20 raise the correct error
- [ ] Bedrock integration test: live API call with `TEST_BEDROCK=true`; single `LLM_CALL` to Claude 3.5 Sonnet with prompt "Say 'hello conductor'"; asserts response contains "hello" (case-insensitive)
- [ ] Checkpoint pause/resume test: run `refactor-dead-code.agent` against mock Bedrock; assert status is `PAUSED` at checkpoint step; call `POST /runs/{id}/approve` with `{decision: "approved"}`; assert run completes with status `COMPLETED`
- [ ] All tests run in `< 60s` in CI (excluding live Bedrock tests gated by `TEST_BEDROCK=true`)
- [ ] GitHub Actions workflow at `.github/workflows/ci.yml` runs the test suite on every push

**Agent context needed**:
- All prior task outputs
- `pytest-asyncio` for async test support
- `respx` or `httpretty` for mocking Bedrock HTTP calls

**Deliverable**: `tests/e2e/`, `tests/fixtures/negative/`, `.github/workflows/ci.yml`

---

## Phase 1 Dependency Graph

```
TASK-109 (Postgres Schema)
    │
    ├──> TASK-110 (Stripe Billing)
    │        │
    │        └──> TASK-111 (FastAPI Backend) ──────────────────────────────┐
    │                     │                                                 │
    │                     └──> TASK-112 (Web IDE Editor)                   │
    │                                  │                                    │
    │                                  └──> TASK-113 (Run Dashboard)       │
    │                                                │                      │
TASK-101 (Lexer)                                    └──> TASK-114 (Auth) ──┤
    │                                                            │           │
TASK-102 (Parser)                                               └─────────> TASK-115 (E2E Tests)
    │
TASK-103 (AST + Task Graph)
    │
TASK-104 (Symbol Table)
    │
TASK-105 (Type Checker)
    │
TASK-106 (IR Emitter)
    │
TASK-107 (Bedrock Dispatcher)
    │
TASK-108 (State Machine + Checkpointing)
    │
    └──> TASK-111 (FastAPI Backend)
```

**Critical path**: TASK-101 → 102 → 103 → 104 → 105 → 106 → 107 → 108 → 111 → 112 → 113 → 114 → 115

TASK-109 and TASK-110 are parallelizable with the compiler chain and should start in Week 1.

---

## Risk Table

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| AWS Bedrock quota limits block integration testing | Medium | High | Request Bedrock quota increase on Day 1; use mocked Bedrock in unit/integration CI; reserve live tests for staging |
| Indentation-sensitive lexer (INDENT/DEDENT) is significantly harder than expected, slipping TASK-101 | Medium | High | Budget 2× the S estimate for lexer if INDENT/DEDENT proves complex; consider switching to brace-delimited syntax if still blocked at Day 3 |
| Stripe metered billing API behavior in test mode differs from production | Low | Medium | Run a full billing end-to-end test in Stripe test mode before launching any paid users; keep billing logic behind a feature flag |
| Clerk JWT validation adds unexpected latency (>100ms per request) | Low | Medium | Benchmark JWT validation in staging; if slow, cache decoded claims in Redis with 60s TTL |
| Web IDE Monaco editor bundle size causes >5s initial load | Medium | Low | Code-split Monaco with dynamic import; lazy-load the `agent` language definition; target <2s LCP on fast connection |
