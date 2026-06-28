---
type: Spec
title: "Conductor Runtime Specification v0.1"
description: Formal specification for the Conductor runtime — IR instruction set semantics, execution state machine, dispatcher contracts per opcode, context-window management, tool executor contracts, idempotency guarantees, and multi-tenant isolation rules.
resource: /startup/05-specs/runtime-spec.md
tags: [runtime, ir, dispatcher, state-machine, context, isolation, spec, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor Runtime Specification v0.1

**Status**: Draft  
**Applies to**: Conductor runtime v0.1.x  
**Conformance suite**: `tests/conformance/runtime/`

---

## 1. IR Format

The Conductor compiler emits an **Intermediate Representation (IR)** as a JSON array of instruction objects. The runtime is an IR walker: it executes instructions sequentially, following `JUMP`, `BRANCH`, and `LOOP_*` instructions to redirect the program counter.

### 1.1 IR Instruction Encoding (JSON)

Each IR instruction is a JSON object with a mandatory `"op"` string field naming the opcode, plus opcode-specific fields. The complete IR for a pipeline run is a flat array under the top-level `"ir"` key:

```json
{
  "ir": [
    {"op": "CALL", "tool": "fs.glob", "args": ["./src/**/*.ts"], "output": "$source_files"},
    {"op": "CONTEXT_PUSH", "sources": [
      {"type": "var", "ref": "$source_files"},
      {"type": "retrieve", "ref": "wiki/concepts/symbol-table.md"}
    ]},
    {"op": "LLM_CALL", "model": "default", "prompt": "Identify unused exports and dead branches. Output JSON: [{file, line, reason}].", "output": "$dead_items"},
    {"op": "CONTEXT_POP"},
    {"op": "CHECKPOINT", "display": "$dead_items", "output": "$checkpoint_result"},
    {"op": "BRANCH", "input": "$checkpoint_result", "field": "status", "cases": [
      {"value": "approved", "target": "L_RefactorEach"},
      {"value": "rejected", "target": "L_Abort"}
    ]},
    {"op": "LABEL", "name": "L_RefactorEach"},
    {"op": "LOOP_START", "list": "$dead_items", "item": "$item", "label": "L_RefactorEach_loop"},
    {"op": "CALL", "tool": "fs.read", "args": ["$item.file"], "output": "$file_content"},
    {"op": "CONTEXT_PUSH", "sources": [
      {"type": "var", "ref": "$file_content"},
      {"type": "var", "ref": "$item.reason"}
    ]},
    {"op": "LLM_CALL", "model": "codegen", "prompt": "Remove dead code at line {{ $item.line }}. Preserve all tests.", "output": "$patched_file"},
    {"op": "CALL", "tool": "fs.write", "args": ["$item.file", "$patched_file.content"], "output": "$write_result"},
    {"op": "CONTEXT_POP"},
    {"op": "LOOP_END", "label": "L_RefactorEach_loop"},
    {"op": "CALL", "tool": "shell.run", "args": ["npm test"], "output": "$test_result"},
    {"op": "BRANCH", "input": "$test_result", "field": "exit_code", "cases": [
      {"value": 0, "target": "L_Done"},
      {"value": "__else__", "target": "L_FindDeadCode"}
    ]},
    {"op": "LABEL", "name": "L_Done"},
    {"op": "SUMMARY", "template": "Refactor complete. {{ len($dead_items) }} items removed.", "output": "$final"},
    {"op": "HALT", "message": "$final"},
    {"op": "LABEL", "name": "L_Abort"},
    {"op": "SUMMARY", "template": "User rejected changes. No files written.", "output": "$abort_msg"},
    {"op": "HALT", "message": "$abort_msg"}
  ]
}
```

### 1.2 IR Instruction Semantics

The following table defines the complete instruction set. All `$`-prefixed strings are variable references resolved from the execution frame's variable store. Field paths (e.g., `"$item.file"`) are traversed at runtime using the same rules as the compile-time type checker (Language Spec §4.3, Rule T-FIELD).

| Opcode | Fields | Inputs | Outputs | Side Effects | Failure Modes | Retry Policy |
|---|---|---|---|---|---|---|
| `LLM_CALL` | `model`, `prompt`, `output` | Model alias, assembled context window, rendered prompt | `output` var set to `ModelResponse` | POST to LLM provider API; token count appended to `run_events` | 429 → exponential backoff (2 s, 4 s, 8 s, 16 s) × 4 then FAIL; 500 → immediate retry × 1 then FAIL; network error → retry × 2 then FAIL | Per above |
| `CALL` | `tool`, `args`, `output` | Tool name, evaluated arg list | `output` var set to tool's return value | Tool-specific I/O (filesystem, subprocess, HTTP) | See §5 per tool; any unhandled exception → `TOOL_ERROR` → FAIL | None by default; tool-specific policy in §5 |
| `ASSIGN` | `var`, `value` | Literal JSON value | `var` set to `value` | None | `value` not valid JSON → `ASSIGN_ERROR` | N/A |
| `BRANCH` | `input`, `field`, `cases` | `$input` var, field name, case list | None (mutates program counter) | None | `$input` not found → E001 FAIL; `field` not present → E003 FAIL; no matching case and no `__else__` → `BRANCH_NO_MATCH` FAIL | N/A |
| `LOOP_START` | `list`, `item`, `label` | `$list` var | Initializes loop iterator; sets `$item` for first iteration | Pushes loop frame onto call stack | `$list` not found → E001 FAIL; `$list` not a list type → E002 FAIL; empty list → jump immediately to matching `LOOP_END` | N/A |
| `LOOP_END` | `label` | Loop frame on stack | Advances iterator; jumps back to matching `LOOP_START` (next iteration) or falls through (exhausted) | Pops loop frame when exhausted | Mismatched `label` → `LOOP_MISMATCH` FAIL | N/A |
| `CONTEXT_PUSH` | `sources` | List of source descriptors (`type`: `"var"`, `"file"`, or `"retrieve"`; `ref`: variable name or path) | Pushes assembled context onto context stack | Retrieves files/vars; serializes to token arrays | Budget exceeded → E010 FAIL; retrieval failure → `RETRIEVE_ERROR` FAIL | Retrieval: 1 retry on timeout |
| `CONTEXT_POP` | — | Context stack (must be non-empty) | Pops top context frame | Frees token budget allocated by matching `CONTEXT_PUSH` | Empty stack → `CONTEXT_UNDERFLOW` FAIL (indicates compiler bug) | N/A |
| `RETRIEVE` | `path`, `output` | Path string | `output` var set to `DocumentChunk` | Loads file from bundle; caches 60 s per path per run | Not found → `RETRIEVE_NOT_FOUND` FAIL | 1 retry on I/O timeout |
| `CHECKPOINT` | `display`, `output` | `$display` var | `output` var set to `ApprovalResult` after resume | Emits SSE `checkpoint` event; suspends run (state → PAUSED) | Human timeout → `CHECKPOINT_TIMEOUT` FAIL; run cancelled → `RUN_CANCELLED` FAIL | N/A — human must re-approve |
| `SUMMARY` | `template`, `output` | Template string | `output` var set to rendered `String` | None | Template parse/eval failure → E013 FAIL | N/A |
| `JUMP` | `target` | Label name | None (mutates program counter) | None | Label not found → `LABEL_NOT_FOUND` FAIL | N/A |
| `LABEL` | `name` | — | Registers label at current IR index | None | Duplicate label name → `DUPLICATE_LABEL` (compiler bug) | N/A |
| `HALT` | `message` | `$message` var | — | Sets run status to `COMPLETED`; writes final output to run record | `$message` not found → `HALT_MISSING_VAR` (terminates anyway with empty output) | N/A |

**`BRANCH` `__else__` sentinel**: A case with `"value": "__else__"` matches any value not matched by a preceding case. It must be the last entry in the `cases` array.

---

## 2. Execution State Machine

Each pipeline run is a state machine instance. The current state is stored in the `pipeline_runs` table and every transition is recorded as an append-only row in `run_events`.

### 2.1 States

| State | Description |
|---|---|
| `PENDING` | Run created; IR validated; not yet dispatched to executor |
| `RUNNING` | Executor is actively processing IR instructions |
| `PAUSED` | Execution suspended at a `CHECKPOINT` instruction awaiting human input |
| `COMPLETED` | `HALT` instruction reached; final output written |
| `FAILED` | Unrecoverable error encountered; no further execution |
| `ABORTED` | Manually cancelled via `DELETE /v1/runs/{run_id}` |

### 2.2 Transitions

```
PENDING  --[executor picks up run]-------------> RUNNING
RUNNING  --[CHECKPOINT instruction]------------> PAUSED
PAUSED   --[POST /runs/{id}/approve, approved]-> RUNNING
PAUSED   --[POST /runs/{id}/approve, rejected]-> RUNNING (follows branch to Abort step)
PAUSED   --[CHECKPOINT_TIMEOUT]----------------> FAILED
PAUSED   --[DELETE /runs/{id}]-----------------> ABORTED
RUNNING  --[HALT instruction]------------------> COMPLETED
RUNNING  --[unrecoverable error]--------------> FAILED
RUNNING  --[DELETE /runs/{id}]-----------------> ABORTED
FAILED   --[POST /runs/{id}/retry]-------------> RUNNING (resumes from last checkpoint)
```

**Retry from checkpoint**: When a run is retried after FAILED, it resumes from the IR index of the most recent successfully-committed idempotency key (see §6). It does not re-execute steps that have committed results.

### 2.3 Persistence

**Table `pipeline_runs`** (Postgres):

| Column | Type | Description |
|---|---|---|
| `run_id` | `TEXT PRIMARY KEY` | Random URL-safe string, e.g., `run_abc123` |
| `pipeline_id` | `TEXT` | Foreign key to `pipelines` |
| `status` | `TEXT` | Current state (enum above) |
| `ir_index` | `INT` | Index of the next instruction to execute |
| `var_store` | `JSONB` | Current variable bindings serialized as `{"$var": value, ...}` |
| `context_stack` | `JSONB` | Serialized context stack for resume |
| `created_at` | `TIMESTAMPTZ` | |
| `started_at` | `TIMESTAMPTZ` | |
| `completed_at` | `TIMESTAMPTZ` | |

**Table `run_events`** (Postgres, append-only):

| Column | Type | Description |
|---|---|---|
| `event_id` | `BIGSERIAL PRIMARY KEY` | |
| `run_id` | `TEXT` | Foreign key |
| `event_type` | `TEXT` | `step_started`, `llm_call`, `llm_response`, `tool_call`, `checkpoint`, `step_completed`, `state_transition`, `error` |
| `ir_index` | `INT` | IR instruction index when event occurred |
| `payload` | `JSONB` | Event-specific data (matches SSE event payload shape from API Spec §5) |
| `tokens_input` | `INT` | For `llm_call` events: input token count |
| `tokens_output` | `INT` | For `llm_response` events: output token count |
| `cost_usd` | `NUMERIC(12,6)` | For `llm_response` events: USD cost |
| `created_at` | `TIMESTAMPTZ` | |

---

## 3. Dispatcher Contract

The dispatcher is the component that reads an IR instruction, executes it, and updates the run state. For each instruction, the following contracts are enforced.

### 3.1 `LLM_CALL`

**Preconditions**:
- Model alias is declared in the pipeline config (either explicitly or as `default`)
- Model alias is mapped to a provider endpoint in the run's model config
- The context window has been assembled by a preceding `CONTEXT_PUSH` (or the context stack is empty, meaning no context — valid for zero-context prompts)
- Total token count of assembled context + rendered prompt ≤ declared budget

**Execution**:
1. Render the `prompt` string by evaluating all `{{ expr }}` template expressions against the current var store.
2. Assemble the messages array: `[{"role": "system", "content": system_prompt}, ...context_messages, {"role": "user", "content": rendered_prompt}]` where `system_prompt` is the Conductor system prompt (not exposed in IR).
3. POST to the provider API. Request includes `model`, `messages`, `max_tokens` (derived from budget headroom), and `stream: false`.
4. Await response. Parse `content` from the response body.
5. Assign `ModelResponse{content, model, tokens_used}` to `output` var.
6. Append `llm_call` and `llm_response` events to `run_events`.
7. Deduct `tokens_used` from the run's remaining budget.

**Postconditions**:
- `output` var is set in the var store
- `run_events` contains two new rows (`llm_call`, `llm_response`)
- Budget counter is decremented

**Error handling**:
- HTTP 429 from provider: exponential backoff — wait 2 s, 4 s, 8 s, 16 s between retries (4 total attempts); if all fail, transition run to FAILED.
- HTTP 500 from provider: immediate retry once; if fails again, transition to FAILED.
- Network timeout (default 120 s): treated as HTTP 500 equivalent.
- Malformed response (not parseable as provider JSON schema): transition to FAILED immediately.

### 3.2 `CALL` (tool invocations)

**Preconditions**: Tool name is in the registered tool table; all `args` can be resolved (variable references exist in var store).

**Execution**:
1. Evaluate each element of `args`: resolve `$var` references, traverse field paths.
2. Dispatch to the tool executor (see §5 for per-tool contracts).
3. Assign the return value to `output` var.
4. Append `tool_call` event to `run_events`.

**Postconditions**: `output` var set; `tool_call` event written.

**Error handling**: Per-tool (§5). Unhandled exceptions: transition run to FAILED.

### 3.3 `BRANCH`

**Preconditions**: `$input` var is set; `field` is valid for the input type; at least one case matches or a `__else__` case exists.

**Execution**:
1. Resolve `$input` from var store.
2. Access `field` on `$input`.
3. Iterate `cases` in order. First case where `value == field_value` sets the program counter to the instruction index of the matching `LABEL`.
4. If no case matches and `__else__` is absent, transition to FAILED with `BRANCH_NO_MATCH`.

**Postconditions**: Program counter points to the first instruction after the matched `LABEL`.

### 3.4 `LOOP_START` / `LOOP_END`

**`LOOP_START` Preconditions**: `$list` var is set and is a list type.

**`LOOP_START` Execution**:
1. Push a loop frame onto the loop stack: `{list: $list, index: 0, item_var: "$item", label: label}`.
2. If `$list` is empty, jump immediately to the matching `LOOP_END` instruction (fall-through).
3. Otherwise, set `$item` in the var store to `$list[0]`.

**`LOOP_END` Execution**:
1. Increment the loop frame's `index`.
2. If `index < len($list)`, set `$item = $list[index]` and jump to the IR index immediately after the matching `LOOP_START`.
3. If `index == len($list)`, pop the loop frame and fall through (loop exhausted).

---

## 4. Context Window Management

The context window is a stack of token sequences. Each `CONTEXT_PUSH` instruction adds one frame; the matching `CONTEXT_POP` removes it. The current context window presented to an `LLM_CALL` is the concatenation of all frames on the stack.

### 4.1 Token Counting

Token counting is performed at `CONTEXT_PUSH` time, before the instruction commits:

- **Bedrock Claude models** (`claude-3-5-sonnet`, `claude-3-haiku`, etc.): use `tiktoken` with `cl100k_base` encoding.
- **Gemma4 models** (self-hosted vLLM): use the model's tokenizer loaded from the vLLM endpoint's `/tokenize` API.
- **GPT-4o** (OpenAI): use `tiktoken` with `o200k_base` encoding.
- **Llama3 / Mistral**: use `tiktoken` with `llama3` or `mistral` vocabulary respectively.

The token count for a `CONTEXT_PUSH` frame is:

```
frame_tokens = sum(tokenize(serialize(source)) for source in sources)
               + len(sources) * 4  # per-message overhead
```

### 4.2 Budget Enforcement

Let `B` = the declared `context budget` (in tokens), and `U` = total tokens across all current context stack frames.

- At each `CONTEXT_PUSH`: compute `frame_tokens`. If `U + frame_tokens > B`, apply the compression policy (§4.3) before pushing.
- At each `LLM_CALL`: assert `U + prompt_tokens ≤ B`. If the assertion fails, transition to FAILED with error E010.

### 4.3 Compression and Eviction Policy

When a `CONTEXT_PUSH` would exceed the budget:

1. **Compression (first pass)**: The runtime inserts an implicit `LLM_CALL` targeting the `default` model with a summarization prompt against the oldest (bottom-of-stack) context frame. The frame is replaced with the summary. This operation consumes tokens from the budget.

2. **Eviction (second pass)**: If compression is insufficient, the oldest frames are dropped from the bottom of the stack (LIFO eviction) until the budget is satisfied. Eviction is logged as a `context_eviction` event in `run_events`.

3. **Hard failure**: If eviction would remove frames required by the current `LLM_CALL`'s explicit `context:` references (as encoded in the IR), the runtime transitions to FAILED with E010 and reports total token usage in the error payload.

---

## 5. Tool Executor Contracts

All tool executors share the following baseline behavior:
- Arguments are fully evaluated (all variable references resolved) before the executor is invoked.
- The result is written to the `output` variable only after the executor returns successfully.
- Executors are synchronous from the IR walker's perspective (async I/O is internal to the executor implementation).

### 5.1 `fs.glob`

- **Implementation**: Python `glob.glob(pattern, recursive=True)` executed in the run's workspace directory.
- **Timeout**: 30 s. If exceeded: `TOOL_TIMEOUT`.
- **Empty result**: Returns `[]` — not an error.
- **Max results**: 10,000 files. If exceeded, returns first 10,000 and logs a warning event.

### 5.2 `fs.read`

- **Max file size**: 10 MiB. Files larger than this limit produce `FILE_TOO_LARGE` and the run transitions to FAILED.
- **Encoding**: Files are read as UTF-8. Non-UTF-8 bytes produce `ENCODING_ERROR`.
- **Timeout**: 10 s for local filesystem reads; 30 s for remote/NFS-mounted paths.

### 5.3 `fs.write`

- **Atomicity**: Write target is `<path>.conductor.tmp` in the same directory; renamed to `<path>` on success. If rename fails (e.g., cross-device), a direct write is performed and the non-atomic nature is recorded in the event log.
- **Directory creation**: `os.makedirs(parent_dir, exist_ok=True)` is called before writing.
- **Idempotency**: Covered by the global idempotency key mechanism (§6).

### 5.4 `shell.run`

- **Isolation**: Each `shell.run` invocation launches a dedicated Docker container. The image is `conductor-sandbox:0.1` (based on `python:3.12-slim`). The container is destroyed after the command completes.
- **Resource limits**: CPU: 1 core. Memory: 512 MiB. PIDs: 64.
- **Network**: Blocked by default (`--network none`). Enabled if `allow_network: true` in `conductor.yaml`.
- **Filesystem**: The run's workspace directory is mounted read-write at `/workspace`. No other host paths are mounted.
- **Timeout**: 300 s default; configurable per-pipeline via `conductor.yaml` `shell_timeout_seconds`.
- **Exit code**: Returned as-is; non-zero exit codes are not errors at the executor level — they are handled by the `condition:` dispatch logic in the calling step.

### 5.5 `human.checkpoint`

- **Suspension**: The IR walker halts at the `CHECKPOINT` instruction and writes state to Postgres. The worker process releases the run and exits cleanly.
- **SSE emission**: Emits `checkpoint` event on the run's SSE stream before suspending.
- **Resume**: A POST to `/v1/runs/{run_id}/approve` loads the run state, sets the `output` var to `ApprovalResult`, and re-enqueues the run.
- **Timeout**: Default 24 h. Configurable via `conductor.yaml` `checkpoint_timeout_seconds`. On timeout, transitions to FAILED.

### 5.6 `retrieve`

- **Source**: Reads from the pipeline bundle (a directory tree uploaded with the pipeline or mounted from the repo root).
- **Cache**: LRU cache keyed by `(run_id, path)`, TTL 60 s. Cache is in-process; not shared across workers.
- **Max document size**: 2 MiB.

---

## 6. Idempotency and Retry

### 6.1 Idempotency Key Computation

Every IR instruction that has side effects receives an idempotency key:

```
idempotency_key = sha256(run_id + ":" + str(ir_index) + ":" + str(attempt_count))
```

Where `attempt_count` starts at `0` and increments on each retry of the same instruction (e.g., after a transient failure).

### 6.2 Key Lifecycle

1. Before executing a side-effecting instruction, the runtime checks the `idempotency_keys` table for `(run_id, ir_index)`.
2. If a committed record exists, the cached result is loaded from the record and the execution is skipped.
3. If no record exists, the instruction is executed. On success, the result is written to `idempotency_keys` atomically with the var store update.
4. On failure, no record is written. The next retry will use `attempt_count + 1` (a different key) to avoid caching failed results.

### 6.3 Non-Idempotent Instructions

`LLM_CALL` is **not** idempotent by design: re-execution produces a fresh LLM response. The idempotency key mechanism is not applied to `LLM_CALL` unless the run config explicitly sets `"cache_llm_calls": true`, which is off by default. When `cache_llm_calls` is enabled, identical prompt + context hashes are used as cache keys and responses are cached for 1 hour.

### 6.4 Side-Effecting Instructions

The idempotency key mechanism is applied to: `CALL` (for `fs.write`, `shell.run`, `http.post`), `CHECKPOINT`, and `RETRIEVE`. It is not applied to `LLM_CALL` (by default), `ASSIGN`, `BRANCH`, `JUMP`, `LABEL`, `LOOP_START`, `LOOP_END`, `CONTEXT_PUSH`, `CONTEXT_POP`, `SUMMARY`, or `HALT`.

---

## 7. Multi-Tenant Execution Isolation

### 7.1 Process Isolation

- Each pipeline run is processed by a single worker goroutine/thread. Workers are assigned from a pool and are not shared between concurrent runs.
- Run state (var store, context stack) is stored in Postgres and never held in worker process memory across suspension boundaries.

### 7.2 Container Isolation (for `shell.run`)

- One Docker container per `shell.run` invocation. Containers are never reused across invocations or runs.
- Containers have no access to the host network (unless `allow_network: true`) and no access to other tenants' filesystem mounts.
- Container lifecycle: created synchronously before `shell.run` dispatch, destroyed synchronously after exit (successful or timed out).
- No persistent filesystem state between steps: if a step produces a file with `fs.write`, the file is written to the shared workspace volume, not inside the container.

### 7.3 Workspace Isolation

- Each run receives a workspace directory at `/workspaces/<run_id>/`. This directory is created at run start and deleted 24 h after run completion.
- Workspaces are on a shared NFS volume with per-tenant quotas (default 1 GiB per tenant, not per run).
- `shell.run` containers mount the run's workspace at `/workspace` (read-write). They cannot access other runs' or tenants' workspace directories.

### 7.4 Secret Management

- Tenant API keys for LLM providers (e.g., Anthropic API key) are stored in AWS Secrets Manager and retrieved at LLM call dispatch time. They are never written to the IR or the var store.
- Secrets are not exposed in SSE events, run status responses, or `run_events` table rows.

---

## 8. Token Budget Accounting

- The declared budget (e.g., `context budget: 32k tokens`) is stored in the run config as `budget_tokens: 32768`.
- A `remaining_budget` counter is initialized to `budget_tokens` at run start.
- Each `CONTEXT_PUSH` decrements `remaining_budget` by `frame_tokens`.
- Each `CONTEXT_POP` increments `remaining_budget` by the popped frame's `frame_tokens`.
- Each `LLM_CALL` decrements `remaining_budget` by `(input_tokens + output_tokens)` from the actual API response.
- If `remaining_budget` drops below zero at any point, the run transitions to FAILED with error E010. The error payload includes `{"tokens_used": N, "budget": B, "overage": N - B}`.
- Token usage is surfaced in the `GET /v1/runs/{run_id}` response as `tokens_total` and in individual step records.
