---
type: Design Doc
title: "Conductor Runtime Orchestrator Design"
description: Stateful IR walker, dispatch table, model router, checkpoint/resume mechanics, context stack, and SSE event stream for the Conductor runtime orchestrator. Implements reliability principles from harness/README.md — checkpointing after every step, idempotency keys on side effects, and hard budget enforcement.
resource: /startup/06-design/runtime-design.md
tags: [runtime, orchestrator, ir-walker, checkpoint, model-router, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor Runtime Orchestrator Design

## 1. Overview

The runtime orchestrator is a **stateful IR walker**. It takes the IR instruction list produced by the compiler and executes it one instruction at a time, persisting all state to Postgres after each step so that any run can be resumed from exactly where it left off after any kind of interruption — crashed container, spot instance eviction, model provider timeout, or human rejection at a checkpoint.

The design is a direct implementation of the reliability principles in `harness/README.md`:

> *If something must happen every time, codify it in the harness — not in a prompt.*

Every mandatory behavior in the runtime (budget checks, idempotency key generation, checkpoint serialization, retry backoff, max iteration enforcement) is deterministic code, not an LLM instruction. The LLM is a bounded component invoked at specific IR instructions; it does not control the execution path.

Key design properties:
- **Checkpoint after every IR instruction** (not only at `human.checkpoint` steps)
- **Idempotency keys** on all `LLM_CALL` and `CALL` side-effecting operations
- **Hard budget enforcement** before each `LLM_CALL`
- **Per-run ECS task isolation** so one pipeline cannot starve or crash others
- **Maximum loop iteration limit** (1000 per `LOOP_START`) enforced in code
- **Exponential backoff retry** on provider errors (not silent same-step retry)
- **Run state fully queryable** from the control plane at any moment

---

## 2. IR Walker Loop

```python
import time
import uuid
from enum import auto, Enum
from dataclasses import dataclass, field
from typing import Optional, Any

MAX_RETRIES = 3
RETRY_BASE_DELAY_S = 2.0
MAX_LOOP_ITERATIONS = 1000
HARD_TIMEOUT_S = 3600  # configurable per pipeline via config_json

class RunStatus(str, Enum):
    PENDING = "pending"
    RUNNING = "running"
    PAUSED = "paused"
    COMPLETED = "completed"
    FAILED = "failed"
    ABORTED = "aborted"

class IRWalker:
    def __init__(self, db: AsyncSession, redis: Redis,
                 model_router: ModelRouter, tool_executor: ToolExecutor):
        self.db = db
        self.redis = redis
        self.model_router = model_router
        self.tool_executor = tool_executor
        self.dispatch_table: dict[str, Callable] = {
            "LLM_CALL":      self.handle_llm_call,
            "CALL":          self.handle_call,
            "ASSIGN":        self.handle_assign,
            "BRANCH":        self.handle_branch,
            "LOOP_START":    self.handle_loop_start,
            "LOOP_END":      self.handle_loop_end,
            "CONTEXT_PUSH":  self.handle_context_push,
            "CONTEXT_POP":   self.handle_context_pop,
            "RETRIEVE":      self.handle_retrieve,
            "CHECKPOINT":    self.handle_checkpoint,
            "SUMMARY":       self.handle_summary,
            "JUMP":          self.handle_jump,
            "LABEL":         self.handle_label,
            "HALT":          self.handle_halt,
        }

    async def execute(self, run: PipelineRun) -> None:
        ir: list[IRInstruction] = run.ir_json
        deadline = run.created_at.timestamp() + (run.config_json.get("timeout_s") or HARD_TIMEOUT_S)

        while run.ir_index < len(ir):
            # Hard timeout
            if time.time() > deadline:
                run.status = RunStatus.FAILED
                run.error_json = {"code": "E_TIMEOUT", "message": "Run exceeded hard timeout"}
                await self.persist_state(run)
                await self.emit_event(run, "run_failed", run.error_json)
                return

            instruction = ir[run.ir_index]
            handler = self.dispatch_table.get(instruction.op)
            if handler is None:
                run.status = RunStatus.FAILED
                run.error_json = {"code": "E_UNKNOWN_OP", "message": f"Unknown IR op: {instruction.op}"}
                await self.persist_state(run)
                return

            try:
                await self.emit_event(run, "step_started", {"op": instruction.op,
                                                            "step": instruction.step_name,
                                                            "ir_index": run.ir_index})
                await handler(run, instruction)

                run.ir_index += 1
                run.retry_count = 0           # reset retry counter on success
                await self.persist_state(run)
                await self.emit_event(run, "step_completed", {"op": instruction.op,
                                                              "step": instruction.step_name,
                                                              "ir_index": run.ir_index})

            except HaltException as e:
                run.status = RunStatus.COMPLETED
                run.output = e.message
                run.completed_at = datetime.utcnow()
                await self.persist_state(run)
                await self.emit_event(run, "run_completed", {"output": e.message})
                return

            except CheckpointException:
                # State already persisted by handle_checkpoint
                return

            except RecoverableError as e:
                run.retry_count += 1
                if run.retry_count <= MAX_RETRIES:
                    delay = RETRY_BASE_DELAY_S * (2 ** (run.retry_count - 1))
                    await self.emit_event(run, "retrying",
                                         {"error": str(e), "attempt": run.retry_count, "delay_s": delay})
                    await self.persist_state(run)
                    await asyncio.sleep(delay)
                    # do NOT increment ir_index; retry the same instruction
                else:
                    run.status = RunStatus.FAILED
                    run.error_json = {"code": "E_MAX_RETRIES", "message": str(e), "ir_index": run.ir_index}
                    await self.persist_state(run)
                    await self.emit_event(run, "run_failed", run.error_json)
                    return

            except BudgetExceededError as e:
                run.status = RunStatus.FAILED
                run.error_json = {"code": "E010", "message": str(e)}
                await self.persist_state(run)
                await self.emit_event(run, "run_failed", run.error_json)
                return

            except FatalError as e:
                run.status = RunStatus.FAILED
                run.error_json = {"code": "E_FATAL", "message": str(e)}
                await self.persist_state(run)
                await self.emit_event(run, "run_failed", run.error_json)
                return
```

---

## 3. Dispatch Table — Handler Descriptions

**`handle_llm_call(run, instr)`**
1. Resolve `model_alias` → full model name via `run.pipeline.model_aliases`.
2. Assemble context window from `run.context_stack` (see Section 7).
3. Check credit balance: if `estimated_input_tokens > run.tenant.credits_balance` → raise `BudgetExceededError`.
4. Generate idempotency key: `SHA256(run_id + ":" + str(ir_index))`.
5. If a `model_calls` row with this idempotency key exists and is complete → skip API call, reuse cached response (handles retry-after-commit case).
6. Call `model_router.call(model_name, messages, idempotency_key)` → `ModelResponse(content, input_tokens, output_tokens, latency_ms)`.
7. Insert `model_calls` row. Deduct credits. Post Stripe usage record (async, non-blocking).
8. Assign `content` to `run.variables[instr.args["output_var"]]`.
9. Emit `llm_response` event with token counts.

**`handle_call(run, instr)`**
1. Look up `tool_name` in `ToolExecutor.tool_registry`.
2. Resolve `$var` references in `args` list from `run.variables`.
3. Generate idempotency key `SHA256(run_id + ":" + str(ir_index) + ":" + tool_name)`.
4. Check idempotency table for existing result.
5. Dispatch to tool handler (see tool-executor details). Capture result.
6. Insert `tool_calls` row with result.
7. Validate result type against expected type from IR (raises `FatalError` on mismatch).
8. Assign to `run.variables[instr.args["output_var"]]`.

**`handle_branch(run, instr)`**
1. Resolve `input_var` from `run.variables`. Extract the `field` (e.g., `"decision"` from a CheckpointDecision).
2. Find matching arm in `cases[]` where `arm.on_value == field_value`.
3. If no arm matches: raise `FatalError("No branch arm matched value '...' in step '...'")`.
4. Set `run.ir_index` to the index of the `LABEL` instruction with name == arm's `target_step`.
   - The LABEL index lookup is pre-computed into a `label_index: dict[str, int]` at run start.
5. Do NOT increment `ir_index` in the main loop (the branch handler already set it); use a special sentinel return.

**`handle_loop_start(run, instr)`**
1. Resolve `list_var` from `run.variables`. Store as `run.loop_state[loop_label] = {"list": [...], "index": 0}`.
2. If list is empty → jump past matching `LOOP_END` (advance `ir_index` past it).
3. Assign `run.variables[item_var] = list[0]`.
4. Push loop scope onto `run.scope_stack`.
5. Increment iteration counter. If > `MAX_LOOP_ITERATIONS` → raise `FatalError("Loop exceeded maximum iterations")`.

**`handle_loop_end(run, instr)`**
1. Advance `loop_state[loop_label].index`.
2. If more items remain: assign next item to `item_var`, set `ir_index` to matching `LOOP_START` label index.
3. If exhausted: pop scope, clean up `loop_state[loop_label]`.

**`handle_context_push(run, instr)`**
1. For each source in `sources[]`:
   - `var`: read string from `run.variables[var_name]`
   - `retrieve`: call `handle_retrieve` inline, cache result
   - `fs_read`: call `fs.read` tool
2. Concatenate all resolved strings into a single context entry string.
3. Push onto `run.context_stack`.
4. Token-count the total stack (see Section 7). If over budget: run SUMMARY op.

**`handle_context_pop(run, instr)`**
Pop top entry from `run.context_stack`.

**`handle_checkpoint(run, instr)`**
1. Serialize `run.variables[display_data_var]` to JSON.
2. Store as `run.variables[output_var] = None` (pending human input).
3. Set `run.status = RunStatus.PAUSED`.
4. Persist state to Postgres (`ir_index` points to this CHECKPOINT instruction — will be re-executed on resume, but idempotency will skip the pause and proceed with the now-populated output_var).
5. Emit `checkpoint` SSE event with serialized display data.
6. Raise `CheckpointException` (exits the walker loop).

**Resume path:** API receives `POST /runs/{run_id}/approve`. Gateway writes `decision` into `variables_json[output_var]`, sets `status=running`, re-enqueues. Walker resumes at same `ir_index`; sees `output_var` already set; treats CHECKPOINT as no-op and increments.

**`handle_retrieve(run, instr)`**
Load file from the pipeline's context bundle (S3 prefix `tenants/{tenant_id}/bundles/{pipeline_id}/{path}`). Cache in `run.variables["_retrieve_cache"][path]`. Assign to `output_var`.

**`handle_halt(run, instr)`**
Raise `HaltException(message=run.variables.get(instr.args["message_var"], ""))`.

**`handle_jump(run, instr)`**
Set `run.ir_index = label_index[instr.args["label"]]`. Do not increment.

**`handle_label(run, instr)`**
No-op at execution time. Labels only serve as jump targets.

**`handle_summary(run, instr)`**
Call cheapest available model (Tier 3) with a summarization prompt over the top context stack entry. Replace the entry with the summary. Assign summary to `output_var`.

---

## 4. Model Router

```
model alias ("default")
    │
    ▼
pipeline model declaration ("claude-3-5-sonnet")
    │
    ▼
tier selection
    ├─ Bedrock-available models → Tier 1 (Bedrock)
    ├─ tenant tier is "builder"/"team" + private model requested → Tier 2 (vLLM 26B)
    └─ free tier or fast model requested → Tier 3 (vLLM 12B)
    │
    ▼
provider client selection
    ├─ Tier 1: boto3 Bedrock client (uses tenant's decrypted API key or Conductor IAM role)
    ├─ Tier 2/3: httpx OpenAI-compatible client (base_url = GPU serving endpoint)
    └─ OpenAI fallback: openai Python SDK (uses tenant's decrypted OpenAI key)
    │
    ▼
retry policy
    3 attempts, exponential backoff (2s, 4s, 8s)
    Retryable: HTTP 429 (rate limit), HTTP 500, HTTP 503, ConnectionError, TimeoutError
    Non-retryable: HTTP 400 (bad request — prompt injection or malformed), HTTP 401 (auth)
    │
    ▼
response normalization
    All providers → ModelResponse(content: str, input_tokens: int, output_tokens: int, latency_ms: int)
```

**Model Name → Tier Mapping**

| Model Name | Tier | Provider |
|---|---|---|
| `claude-3-5-sonnet` | 1 | Bedrock (us.anthropic.claude-3-5-sonnet-20241022-v2:0) |
| `claude-3-haiku` | 1 | Bedrock |
| `llama-3-70b` | 1 | Bedrock (meta.llama3-70b-instruct-v1:0) |
| `mistral-large` | 1 | Bedrock |
| `gpt-4o` | 1 | OpenAI API |
| `gemma4-26b` | 2 | vLLM private GPU |
| `gemma4-12b` | 3 | vLLM fast GPU |
| `default` (alias) | Resolved per pipeline model declaration | — |

---

## 5. Execution State (Postgres)

The `pipeline_runs` table holds the complete, resumable state of every run. No state exists only in memory.

```
pipeline_runs columns relevant to resumption:
  ir_index          INTEGER   -- next instruction to execute (0-based)
  variables_json    JSONB     -- all $var assignments as {name: value}
  context_stack_json JSONB    -- list of context entry strings (most recent last)
  retry_count       INTEGER   -- current retry attempt for current instruction
  status            VARCHAR   -- pending|running|paused|completed|failed|aborted
  error_json        JSONB     -- populated on failure
  config_json       JSONB     -- timeout_s, max_iterations, etc.
```

**Resume Algorithm**
```python
async def resume(self, run_id: UUID) -> None:
    run = await self.db.get(PipelineRun, run_id)
    # Restore in-memory state from DB columns
    run.variables = run.variables_json or {}
    run.context_stack = run.context_stack_json or []
    run.loop_state = run.config_json.get("_loop_state", {})
    # Pre-compute label index for JUMP/BRANCH handlers
    run._label_index = {
        instr.args["name"]: idx
        for idx, instr in enumerate(run.ir_json)
        if instr.op == "LABEL"
    }
    run.status = RunStatus.RUNNING
    await self.persist_state(run)
    await self.execute(run)
```

---

## 6. Reliability Design

These properties are codified in the runtime, not in prompts. Each maps to a principle from `harness/README.md`.

**Checkpoint after every IR step** (harness/README.md §19)
Every call to `self.persist_state(run)` in the main walker loop writes `ir_index + 1` to Postgres before moving on. A container death between step N and step N+1 results in a replay of step N (handled by idempotency).

**Idempotency keys on side effects** (harness/README.md §16)
`LLM_CALL` and `CALL` each generate a deterministic idempotency key from `(run_id, ir_index, op)`. Before executing, the handler checks the `model_calls` or `tool_calls` table for an existing committed row with that key. If found, it reuses the result. This means retrying a failed run never double-bills or double-writes.

**Exponential backoff retry** (harness/README.md §10)
Provider errors (429, 500, 503, ConnectionError) raise `RecoverableError`. The main loop retries up to `MAX_RETRIES=3` times with delays of 2s, 4s, 8s. It does NOT silently retry the same prompt with no delay — that pattern creates thundering herd against rate-limited providers.

**Hard run timeout** (harness/README.md §12)
Default 3600s, configurable in `pipeline_runs.config_json["timeout_s"]`. Checked at the top of every loop iteration. A pipeline that loops indefinitely will be killed and marked failed after the timeout, not left consuming GPU quota.

**Maximum loop iterations** (harness/README.md §21)
`LOOP_START` handler increments a counter in `loop_state`. If counter > 1000, raise `FatalError`. This quarantines runaway loops instead of letting them thrash.

**Budget hard limit** (harness/README.md §12 + billing)
Before each `LLM_CALL`, the handler estimates input token count from the assembled context (tiktoken). If estimated tokens > current credit balance, raise `BudgetExceededError`. The run fails cleanly with error code E010 and a user-readable message, rather than silently failing mid-execution.

**Structured outputs at phase boundaries** (harness/README.md §9)
Tool outputs are validated against the expected type from the IR (from type checker annotations). A `shell.run` that returns something other than `ExecResult` structure raises `FatalError` before the value is assigned to a variable, preventing downstream type confusion.

**Run state queryable from control plane** (harness/README.md §20)
The `GET /runs/{run_id}` endpoint returns `ir_index`, `status`, `retry_count`, `total_tokens`, `total_cost_usd`, and the last 20 `run_events` rows in a single response. A human or supervisor can always inspect the exact current state of a run.

---

## 7. Context Stack

The context stack is a list of context entries, each being a list of resolved content strings. It lives in `run.context_stack_json` (serialized to Postgres on every IR step).

**Assembly for LLM_CALL**
```python
def assemble_messages(self, run: PipelineRun, prompt_template: str) -> list[dict]:
    # Flatten all context stack entries into a single content block
    context_parts = []
    for entry in run.context_stack:
        context_parts.extend(entry)

    system_content = "\n\n---\n\n".join(context_parts) if context_parts else None
    user_content = self.render_template(prompt_template, run.variables)

    messages = []
    if system_content:
        messages.append({"role": "system", "content": system_content})
    messages.append({"role": "user", "content": user_content})
    return messages
```

**Token Counting and Budget Enforcement**
```python
def count_tokens(self, messages: list[dict]) -> int:
    # Use tiktoken cl100k_base as a provider-agnostic approximation
    enc = tiktoken.get_encoding("cl100k_base")
    total = 0
    for msg in messages:
        total += len(enc.encode(msg["content"])) + 4  # role overhead
    return total + 3  # conversation overhead
```

If `count_tokens(messages) > pipeline.budget_tokens`:
1. Find the oldest context stack entry.
2. Issue a `SUMMARY` op (inline, not as a separate IR instruction) using the cheapest Tier 3 model.
3. Replace the oldest entry with the summary string.
4. Repeat until within budget or only the current user message remains (at that point, raise `BudgetExceededError`).

---

## 8. SSE Event Stream

Each run has a dedicated Redis pub/sub channel: `run:{run_id}:events`.

**Event Schema**
```json
{
  "event_type": "llm_response",
  "run_id": "uuid",
  "ir_index": 7,
  "step_name": "FindDeadCode",
  "timestamp": "2026-06-28T12:34:56.789Z",
  "data": {
    "model": "claude-3-5-sonnet",
    "provider": "bedrock",
    "input_tokens": 4200,
    "output_tokens": 312,
    "latency_ms": 1840
  }
}
```

**Event Types**

| Event | Trigger | Key Data |
|---|---|---|
| `run_started` | Walker begins | `run_id`, `pipeline_name` |
| `step_started` | Before each instruction | `op`, `step_name`, `ir_index` |
| `llm_call` | Before model API call | `model`, `provider`, `estimated_tokens` |
| `llm_response` | After model API call | `input_tokens`, `output_tokens`, `latency_ms` |
| `tool_call` | Before tool dispatch | `tool_name`, `args` (sanitized) |
| `tool_result` | After tool handler returns | `result_type`, `latency_ms` |
| `checkpoint` | At CHECKPOINT instruction | `display_data` (full JSON for UI render) |
| `step_completed` | After each instruction | `op`, `step_name`, `ir_index` |
| `retrying` | On RecoverableError | `error`, `attempt`, `delay_s` |
| `run_completed` | On HALT or last instruction | `output`, `total_tokens`, `total_cost_usd` |
| `run_failed` | On any unrecoverable error | `code`, `message`, `ir_index` |

**FastAPI SSE Endpoint**
```python
@router.get("/runs/{run_id}/events")
async def stream_run_events(run_id: UUID, tenant_id: UUID = Depends(get_tenant)):
    # Verify tenant owns this run
    run = await db.get(PipelineRun, run_id)
    if run.tenant_id != tenant_id:
        raise HTTPException(403)

    async def event_generator():
        # First, replay persisted events from DB (for reconnect)
        events = await db.scalars(
            select(RunEvent).where(RunEvent.run_id == run_id).order_by(RunEvent.created_at)
        )
        for event in events:
            yield f"data: {event.data_json}\n\n"

        # Then subscribe to live events
        async with redis.pubsub() as ps:
            await ps.subscribe(f"run:{run_id}:events")
            async for message in ps.listen():
                if message["type"] == "message":
                    yield f"data: {message['data']}\n\n"
                    if json.loads(message["data"])["event_type"] in ("run_completed", "run_failed"):
                        break

    return EventSourceResponse(event_generator())
```

The replayed DB events on reconnect ensure the Web IDE always shows the full run history even if the user refreshes mid-run or the SSE connection drops.
