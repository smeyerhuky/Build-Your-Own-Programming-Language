---
type: RFC
title: "RFC-001: Conductor Agent Harness Architecture"
description: Proposes the full architecture for the Conductor compiled DSL — from .agent file syntax through lexer, parser, AST, symbol table, type checker, IR emitter, and runtime dispatcher — including multi-tenant isolation, GPU serving, billing, and MCP integration.
resource: /startup/03-rfc/rfc-001-agent-harness.md
tags: [rfc, architecture, compiler, runtime, dsl, agent, harness]
timestamp: 2026-06-28T00:00:00Z
---

# RFC-001: Conductor Agent Harness Architecture

| Field | Value |
|-------|-------|
| RFC Number | 001 |
| Title | Conductor Agent Harness Architecture |
| Author | Founder |
| Status | DRAFT |
| Created | 2026-06-28 |
| Last Updated | 2026-06-28 |
| Depends On | — |
| Supersedes | — |

---

## Abstract

This RFC proposes the complete technical architecture for **Conductor** — a compiled domain-specific language for agentic coding workflows. Conductor enables developers to write declarative `.agent` files describing multi-step, multi-model, multi-tool pipelines; a compiler translates these files into a flat IR instruction list; a runtime dispatcher executes each instruction against real LLM APIs and tool handlers, producing a reproducible, auditable execution trace.

The architecture spans eight tightly coupled subsystems: (1) a tokenizer and recursive-descent parser producing a Task Graph AST, (2) a symbol table with lexical scoping for pipeline variables, (3) a structural type checker against a tool signature registry, (4) a Three-Address Code IR emitter, (5) a runtime dispatcher with execution state machine and checkpoint protocol, (6) a multi-tenant execution sandbox, (7) a GPU serving layer with speculative decoding, and (8) a Stripe-backed billing engine with Bedrock pass-through. Each subsystem is mapped to the corresponding chapter of the *Build Your Own Programming Language* book to leverage the existing compiler theory foundation in `/wiki/`.

---

## Motivation

### Current State of Agent Workflow Tooling

The dominant pattern for agent workflows today is imperative Python orchestration: developers write chains of `llm.invoke()` calls, manually thread output variables from one step to the next, catch exceptions in ad-hoc try/except blocks, and copy API keys into environment variables without any systematic secret management. This pattern has four compounding failure modes:

1. **No type safety.** A tool that returns `{"result": [...]}` is consumed by downstream code that assumes `{"items": [...]}`. The mismatch is discovered at runtime, mid-pipeline, after an expensive LLM call has already been made and billed.
2. **No portability.** A LangChain chain is LangChain code. Migrating from `ChatOpenAI` to `ChatAnthropic` requires rewriting the chain, not changing one line in a config file.
3. **No auditability.** The execution trace lives in log lines if you remembered to add them. Reproducing a specific run to debug a bad output requires manually re-running with the same inputs and hoping the model produces the same intermediate steps.
4. **No composability.** Two pipelines cannot be combined without copy-pasting code. There is no module system, no import mechanism, no shared type definitions.

Developers working at this layer are experienced enough to feel the pain but not resourced enough to build the tooling themselves. This is the classic platform opportunity.

### Why a Compiled DSL Is the Right Abstraction

A compiled DSL addresses all four failure modes at the right level of abstraction:

- **Type safety** is enforced at compile time by checking tool argument types and output field access against a registry of known tool signatures. Errors surface before any API call is made.
- **Portability** is achieved because the `.agent` file declares intent (`model: claude-3-5-sonnet`), not provider SDK calls. The compiler and runtime map model names to provider endpoints. Switching providers is a one-line change in the pipeline file.
- **Auditability** is built into the IR execution model. Every instruction execution is a discrete event that can be logged, replayed, and diffed. The flat IR list is the execution plan; the execution trace is the log of IR[n] → output pairs.
- **Composability** follows from the grammar: pipelines are first-class named entities. A future `import` keyword brings another `.agent` file's named steps into scope, exactly as modules work in any compiled language.

The compiled DSL model also enables static analysis impossible in imperative Python: dead step elimination, cycle detection in the task graph, context budget estimation before execution, and compile-time cost estimation.

### Why the Book's Compiler Pipeline Is the Right Foundation

The *Build Your Own Programming Language* compiler pipeline (Ch3–Ch9/11) provides a battle-tested, chapter-by-chapter implementation path for exactly the components Conductor needs. Rather than building a bespoke parser or reinventing IR design, we follow the established architecture: lexer → parser → AST → symbol table → type checker → IR → runtime. Each chapter maps to one Conductor subsystem, ensuring the implementation is grounded in proven compiler theory rather than ad-hoc design decisions. See `/wiki/` for the theoretical foundation.

---

## Proposed Architecture

### Section A: Language Design

#### .agent File Format

Conductor source files use the `.agent` extension. A file may contain one or more named `pipeline` definitions. The syntax is indentation-significant with colon-terminated blocks, inspired by YAML's readability but extended with control flow constructs that YAML cannot express cleanly.

```agent
pipeline refactor_session:
  model default: claude-3-5-sonnet
  model codegen: gpt-4o
  context budget: 32000 tokens

  step scan_files:
    context:
      retrieve: "./src/**/*.ts"
    prompt: |
      Identify all dead code items in {{$context}}.
      Return JSON: {"dead_items": [...]}
    output: $dead_items

  step review_gate:
    tool: human.checkpoint(data=$dead_items)
    output: $approval

  step apply_fixes:
    loop:
      for $item in $dead_items.items:
        model: codegen
        prompt: |
          Remove dead code item {{$item.name}} from {{$item.file}}.
          Return the corrected file content only.
        tool: fs.write(path=$item.file, content=$response)

  step verify:
    tool: shell.run(cmd="npm test")
    output: $test_result
    branch:
      - on: $test_result.exit_code == 0 -> done
      - on: $test_result.exit_code != 0 -> fail
```

#### Pipeline and Step Primitives

A **pipeline** is the top-level compilation unit. It has a name, optional model declarations, an optional context budget, and an ordered sequence of steps.

A **step** is the fundamental execution unit. Each step may contain exactly one of: an LLM prompt declaration, a tool call, a branch, a loop, or a condition. Steps execute sequentially by default; data flows between steps via `$variables`.

#### Context Declaration Syntax

Context is declared inside a step with the `context:` block. Sources are listed as key-value pairs:

```agent
context:
  retrieve: "./docs/**/*.md"
  file: $previously_read_content
```

The context manager assembles all declared sources into a single context window for the step's LLM call, respecting the pipeline-level `context budget` ceiling.

#### Tool Call Syntax

Tool calls use dot-notation with named arguments:

```agent
tool: fs.glob(pattern="./src/**/*.ts")
output: $source_files

tool: shell.run(cmd="npm test")
output: $test_result
```

Tool arguments may be string literals, `$variable` references, or template expressions `{{ $var.field }}`.

#### Variable Scoping

Variables are prefixed with `$` and follow lexical scoping rules:

- **PipelineScope**: Variables declared at the pipeline level (`output: $var` in any step) are visible to all subsequent steps.
- **LoopScope**: The loop iteration variable (`for $item in $list`) is scoped to the loop body and shadows any outer `$item` definition (which is flagged as a warning, not an error).
- **Step-local**: A variable declared only within a step's `output:` is visible downstream but not upstream.

Variable resolution is strictly forward-only within a pipeline (no forward references), except within loop bodies which may reference variables from outer scopes.

#### Branch and Condition Syntax

```agent
branch:
  - on: $result.exit_code == 0 -> pass_step
  - on: $result.exit_code != 0 -> fail_step
  - on: default -> unknown_step
```

Conditions in `on:` clauses are field-access expressions: `$var.field <op> <literal>`. Supported operators: `==`, `!=`, `>`, `<`, `>=`, `<=`, `contains`, `matches`. The `default` keyword is a catch-all. All branches must be exhaustive (the type checker enforces this for enum-typed fields).

#### Loop Syntax

```agent
loop:
  for $item in $list_var:
    prompt: "Process {{$item.name}}"
    output: $item_result
    tool: fs.write(path=$item.file, content=$item_result)
```

Loop bodies may contain any step constructs except nested pipeline definitions. The IR emitter wraps loop bodies in `LOOP_START` / `LOOP_END` instructions with a back-edge in the task graph.

#### Model Routing Syntax

Model declarations at the pipeline level define named model slots:

```agent
model default: claude-3-5-sonnet
model codegen: gpt-4o
model fast: gemma4-12b
```

Steps reference model slots by name (`model: codegen`). If no `model:` is declared in a step, the `default` slot is used. The runtime's model router resolves slot names to provider endpoints and applies tier selection logic at dispatch time.

#### Context Budget Declaration

```agent
context budget: 32000 tokens
```

The context manager enforces this ceiling per step. If assembled context exceeds the budget, the eviction policy (oldest-first, then lowest-relevance) trims sources until the budget is satisfied. If the prompt itself exceeds the budget, compilation fails with `ContextBudgetExceeded`.

#### Template Interpolation

`{{ $var }}` and `{{ $var.field }}` are evaluated at runtime immediately before an LLM call or tool argument construction. Templates are resolved against the current variable environment. Undefined variables in templates are a compile-time error (caught by the symbol table resolver).

---

### Section B: Compiler Pipeline

#### Lexer (Ch3) — `compiler/lexer.py`

The lexer is a regex-based tokenizer that consumes `.agent` source text and emits a flat list of `(category, lexeme, line, col)` tuples. It makes a single left-to-right pass with no lookahead.

**Token Categories:**

| Category | Examples | Pattern |
|----------|----------|---------|
| `KEYWORD` | `pipeline`, `step`, `model`, `context`, `tool`, `prompt`, `output`, `branch`, `loop`, `condition`, `on`, `retrieve`, `summary` | Keyword set match |
| `IDENT` | `refactor_session`, `scan_files` | `[a-z_][a-z0-9_]*` |
| `DOLLAR_IDENT` | `$dead_items`, `$item` | `\$[a-z_][a-z0-9_.]*` |
| `ARROW` | `->` | `->` |
| `TEMPLATE` | `{{ $item.name }}` | `\{\{[^}]+\}\}` |
| `STRING` | `"./src/**/*.ts"` | `"[^"]*"` |
| `INT` | `32000` | `[0-9]+` |
| `COLON` | `:` | `:` |
| `INDENT` | (indentation increase) | Indentation delta tracking |
| `DEDENT` | (indentation decrease) | Indentation delta tracking |
| `NEWLINE` | line terminator | `\n` |
| `PIPE_LITERAL` | `\|` block scalar start | `\|` |
| `COMMENT` | `# ...` | `#[^\n]*` (discarded) |

Indentation is tracked with a stack of indent levels. Each increase in indentation emits an `INDENT` token; each decrease emits one `DEDENT` per level closed. This produces a token stream that the parser can treat as explicit block delimiters without tracking whitespace.

The lexer is implemented as a compiled regex alternation (the keyword pattern is checked before `IDENT` to prevent keyword shadowing). Approximate token count for the canonical `refactor_session.agent`: 112 tokens.

#### Parser (Ch4) — `compiler/parser.py`

The parser is a hand-written recursive-descent parser consuming the token stream from the lexer. It produces a concrete syntax tree (parse tree) that is immediately lowered to the AST in the same pass.

**Grammar (abbreviated CFG):**

```
pipeline_file    → pipeline_def* EOF
pipeline_def     → 'pipeline' IDENT ':' NEWLINE INDENT pipeline_body DEDENT
pipeline_body    → (model_decl | context_budget | step_def)*
model_decl       → 'model' IDENT ':' IDENT NEWLINE
context_budget   → 'context' 'budget' ':' INT 'tokens' NEWLINE
step_def         → 'step' IDENT ':' NEWLINE INDENT step_body DEDENT
step_body        → (model_ref | context_decl | prompt_decl | tool_call
                  | branch | loop | condition | output_decl)*
model_ref        → 'model' ':' IDENT NEWLINE
context_decl     → 'context' ':' NEWLINE INDENT context_source+ DEDENT
context_source   → ('retrieve' | 'file') ':' (STRING | DOLLAR_IDENT) NEWLINE
prompt_decl      → 'prompt' ':' (STRING | PIPE_LITERAL block_string) NEWLINE
tool_call        → 'tool' ':' IDENT '.' IDENT '(' arg_list ')' NEWLINE
arg_list         → (IDENT '=' (STRING | DOLLAR_IDENT | TEMPLATE))*
output_decl      → 'output' ':' DOLLAR_IDENT NEWLINE
branch           → 'branch' ':' NEWLINE INDENT branch_arm+ DEDENT
branch_arm       → '-' 'on' ':' condition_expr ARROW IDENT NEWLINE
loop             → 'loop' ':' NEWLINE INDENT loop_header step_body DEDENT
loop_header      → 'for' DOLLAR_IDENT 'in' DOLLAR_IDENT ':' NEWLINE
condition_expr   → DOLLAR_IDENT '.' IDENT op (STRING | INT) | 'default'
op               → '==' | '!=' | '>' | '<' | '>=' | '<='
```

The parser reports the first syntax error with line/column information and attempts recovery by consuming to the next `NEWLINE` + `DEDENT` boundary, allowing multiple errors per compilation unit.

#### AST (Ch5) — `compiler/ast_nodes.py`

The parser lowers the parse tree directly to a **Task Graph** — a directed graph where nodes are AST node objects and edges represent dependencies and control flow.

**Node Types:**

| Node | Fields |
|------|--------|
| `PipelineNode` | `name`, `model_slots: Dict[str, str]`, `context_budget: int`, `steps: List[StepNode]` |
| `StepNode` | `name`, `model_slot`, `context_sources`, `prompt_template`, `tool_call`, `output_var`, `branch`, `loop`, `condition` |
| `ToolCallNode` | `tool_name: str`, `method: str`, `args: Dict[str, Expr]`, `output_var` |
| `ContextDeclNode` | `sources: List[ContextSource]` |
| `PromptDeclNode` | `template: str`, `output_var` |
| `BranchNode` | `input_expr`, `arms: List[BranchArm]` |
| `LoopNode` | `list_var`, `item_var`, `body: List[StepNode]` |
| `ConditionNode` | `expr`, `true_branch`, `false_branch` |
| `OutputDeclNode` | `var_name` |

**Edge Types:**

| Edge | Meaning |
|------|--------|
| `DataEdge(src, dst, var)` | `dst` reads `$var` produced by `src` |
| `ControlEdge(src, dst)` | `dst` executes after `src` in a branch arm |
| `BackEdge(loop_end, loop_start)` | Loop cycle; detected and annotated separately |

Back-edges are identified during graph construction by DFS cycle detection. They are preserved in the AST (not removed) and used by the IR emitter to generate `LOOP_START` / `LOOP_END` pairs.

#### Symbol Table (Ch6) — `compiler/symbol_table.py`

The symbol table resolves every `$variable` reference to its declaration site and detects undeclared, shadowed, and unused variables.

**Scope Chain Design:**

```
PipelineScope
  └── StepScope (per step)
        └── LoopScope (per loop, if present)
```

The resolver performs two passes over the AST:
1. **Declaration pass**: Walk the AST in execution order; record every `output: $var` into the current scope.
2. **Reference pass**: For every `$var` use (in template, tool arg, branch condition), walk up the scope chain until found. If not found: `UndefinedVariable` error with the line number.

**Error Codes:**

| Code | Message |
|------|--------|
| `SYM001` | `UndefinedVariable: $var is not declared before use at line N` |
| `SYM002` | `ShadowedVariable: $item shadows outer scope $item declared at line N` (warning) |
| `SYM003` | `UnusedVariable: $var is declared at line N but never referenced` (warning) |

#### Type Checker (Ch7) — `compiler/type_checker.py`

The type checker enforces structural typing against a **Tool Signature Registry** — a hardcoded dict of tool names to their argument and return types.

**Tool Signature Registry:**

| Tool | Arguments | Return Type |
|------|-----------|-------------|
| `fs.glob` | `pattern: String` | `FileList` |
| `fs.read` | `path: String` | `FileContent { content: String, path: String }` |
| `fs.write` | `path: String, content: String` | `WriteResult { success: Bool }` |
| `shell.run` | `cmd: String` | `ExecResult { stdout: String, stderr: String, exit_code: Int }` |
| `human.checkpoint` | `data: Any` | `ApprovalResult { status: "approved"\|"rejected", notes: String }` |
| `retrieve` | `path: String` | `DocumentChunk { content: String, metadata: Object }` |
| `http.get` | `url: String` | `HttpResponse { body: String, status: Int }` |
| `summary` | `template: String` | `String` |

**Type Inference Rules:**

- LLM call outputs (`output: $var` after a `prompt:`) have type `JSON` (opaque; field access is unchecked in v1).
- Tool call outputs have the type from the registry.
- Template `{{ $var.field }}` access is checked: if `$var` has a known struct type and `field` is not in that type's fields, emit `TypeMismatch`.
- Branch condition operands: `$var.field` must resolve to a type that supports the comparison operator (e.g., `Int` for `>`, `String` for `contains`).

**Type Error Codes:**

| Code | Message |
|------|--------|
| `TY001` | `TypeMismatch: $var.exit_code has type String, expected Int` |
| `TY002` | `MissingField: $var.items does not exist on type FileList` |
| `TY003` | `UnknownTool: fs.move is not in the tool registry` |
| `TY004` | `ArgumentTypeMismatch: fs.write expects content:String, got FileList` |

#### IR Emitter (Ch8) — `compiler/ir_emitter.py`

The IR emitter performs a depth-first walk of the Task Graph and emits a flat, sequential list of Three-Address Code (TAC) instructions. The IR is the artifact handed to the runtime; it contains no AST structure.

**Instruction Set:**

| Instruction | Operands | Semantics |
|-------------|----------|-----------|
| `LLM_CALL` | `model, prompt_template, context_vars[], output_var` | Invoke LLM; store JSON response in `output_var` |
| `CALL` | `tool_name, args{}, output_var` | Invoke tool handler; store result in `output_var` |
| `ASSIGN` | `var, value` | Bind `var` to literal or computed `value` |
| `BRANCH` | `input_var, field, cases:{value→label}[]` | Jump to label matching `input_var.field` value |
| `LOOP_START` | `list_var, item_var` | Begin iteration; push loop frame |
| `LOOP_END` | — | Pop loop frame; jump back to `LOOP_START` if items remain |
| `CONTEXT_PUSH` | `sources:[{type, ref}]` | Push context sources onto context stack |
| `CONTEXT_POP` | — | Pop context sources |
| `RETRIEVE` | `path, output_var` | Load and chunk document at `path` |
| `CHECKPOINT` | `display_data_var, output_var` | Pause for human approval; store `ApprovalResult` |
| `SUMMARY` | `template, output_var` | Invoke summary tool; store result |
| `LABEL` | `name` | Branch target; not executed |

Back-edges (loop cycles) are emitted as `LOOP_START` / `LOOP_END` pairs. The emitter tracks the current loop depth and emits `CONTEXT_PUSH` / `CONTEXT_POP` pairs around `LOOP_START` / `LOOP_END` when the loop body declares context sources.

The IR for the canonical `refactor_session.agent` is 28 instructions.

---

### Section C: Runtime Dispatcher

#### IR Walker Loop — `runtime/dispatcher.py`

The runtime maintains a program counter `pc` initialized to 0. The main loop:

```python
while pc < len(ir):
    instr = ir[pc]
    handler = DISPATCH_TABLE[instr.op]
    result = handler(instr, state)
    state.apply(result)
    pc = result.next_pc  # default: pc + 1
```

The dispatch table maps instruction opcodes to async handler coroutines. The entire walker is async (`asyncio`); all handlers `await` their underlying I/O.

#### Per-Instruction Dispatch

| Opcode | Handler | Notes |
|--------|---------|-------|
| `LLM_CALL` | `handle_llm_call` | Checks credit balance first; routes to model router; records usage after |
| `CALL` | `handle_tool_call` | Dispatches to Tool Executor by tool name prefix |
| `ASSIGN` | `handle_assign` | Purely in-memory; no I/O |
| `BRANCH` | `handle_branch` | Evaluates field access; sets `pc` to target label's index |
| `LOOP_START` | `handle_loop_start` | Pushes loop frame: `{list, index: 0, loop_start_pc}` |
| `LOOP_END` | `handle_loop_end` | Increments index; jumps back or falls through |
| `CONTEXT_PUSH` | `handle_context_push` | Calls Context Manager |
| `CONTEXT_POP` | `handle_context_pop` | Calls Context Manager |
| `RETRIEVE` | `handle_retrieve` | Calls retrieval tool handler |
| `CHECKPOINT` | `handle_checkpoint` | Writes pause record to DB; suspends coroutine |
| `SUMMARY` | `handle_summary` | Calls summary tool handler |

#### Model Routing — `runtime/model_router.py`

The model router resolves a `model_slot` name (e.g., `codegen`) to a provider endpoint using a two-step lookup:

1. **Slot resolution**: Look up the slot name in the pipeline's `model_slots` dict → get model string (e.g., `gpt-4o`).
2. **Tier selection**: Match the model string against the tier registry:
   - Pattern `claude-*` → Tier 1 (AWS Bedrock, us-east-1/us-west-2)
   - Pattern `gpt-*` or `o[0-9]-*` → Tier 1 (OpenAI API, user-supplied key)
   - Pattern `gemma4-26b*` → Tier 2 (Private GPU, internal endpoint)
   - Pattern `gemma4-12b*` → Tier 3 (Fast GPU, internal endpoint)
   - Default → reject with `UnknownModel` error

The router applies tenant tier permissions before dispatch: a Free-tier tenant attempting to invoke a `claude-*` model receives `TierAccessDenied`.

**Retry policy**: Exponential backoff with jitter, max 3 retries, on `RateLimitError` or `ServiceUnavailableError`. After 3 failures: escalate to `FAILED` state and surface to user.

#### Tool Executor — `runtime/tool_executor.py`

Tool handlers are organized by namespace prefix:

| Prefix | Handler Class | Notes |
|--------|--------------|-------|
| `fs.*` | `FsHandler` | Reads/writes within tenant S3 prefix; no local filesystem access |
| `shell.*` | `ShellHandler` | Spawns per-tenant Docker container; enforces CPU/memory limits |
| `human.*` | `HumanHandler` | Writes CHECKPOINT record; suspends until UI approval |
| `http.*` | `HttpHandler` | Egress via allow-listed URLs only (configurable per tenant) |
| `retrieve` | `RetrieveHandler` | Vector search over tenant document store |

Each handler receives the fully-resolved argument dict (all `$var` references already substituted by the runtime) and returns a typed result dict matching the tool's registry signature.

#### Context Budget Manager — `runtime/context_manager.py`

The Context Manager maintains a stack of context frames (pushed/popped by `CONTEXT_PUSH`/`CONTEXT_POP`). Before each `LLM_CALL`:

1. **Assembly**: Concatenate all sources from the current frame stack (outermost to innermost).
2. **Token counting**: Call `tiktoken.count_tokens(assembled_context + prompt_template)`.
3. **Enforcement**: If total > `context_budget`, apply eviction:
   - First: truncate `retrieve` results (document chunks are split; emit the highest-scoring chunks only).
   - Second: truncate `file` sources from oldest to newest.
   - Never truncate the `prompt_template` itself.
4. **Compression**: If still over budget after eviction, call the `summary` tool on the largest remaining source to compress it in-place.

The budget ceiling is a hard cap: if assembly cannot fit under budget even after compression, emit `ContextBudgetExceeded` at runtime and transition to `FAILED`.

#### Execution State Machine

```
QUEUED → RUNNING → PAUSED (on CHECKPOINT)
                 → DONE
                 → FAILED
PAUSED → RUNNING (on human approval)
PAUSED → CANCELLED (on human rejection or timeout)
```

State and the current `pc` value are persisted to Postgres after every IR instruction completes. On resume (after a CHECKPOINT approval), the runtime reconstructs the variable environment from the stored state and restarts the walker at `pc + 1` from the checkpoint instruction.

#### Checkpoint Protocol

1. Runtime reaches `CHECKPOINT` instruction.
2. Serializes `{run_id, pc, variable_env, display_data}` to Postgres `runs.checkpoint_state`.
3. Posts event `{type: "PAUSED", run_id, display_data}` to Redis pub/sub.
4. Web IDE subscribes to the event stream and renders the approval UI.
5. User clicks Approve/Reject → POST `/runs/{run_id}/approve` or `/runs/{run_id}/reject`.
6. Control plane API writes `ApprovalResult` to variable env and transitions state to `RUNNING`.
7. Runtime worker picks up the run from the queue and resumes from `pc + 1`.

**Idempotency**: Each `CHECKPOINT` instruction has an `idempotency_key = run_id + ":" + pc`. Duplicate approval requests for the same key are rejected with 409.

#### Error Handling

| Error Type | Recovery Action |
|-----------|----------------|
| `RateLimitError` | Retry with exponential backoff (max 3×) |
| `ToolTimeoutError` | Retry once; then escalate to `FAILED` |
| `ContextBudgetExceeded` | Attempt compression; if still over, `FAILED` |
| `TypeMismatch` (runtime) | `FAILED` immediately (should have been caught at compile time) |
| `CheckpointRejected` | Transition to `CANCELLED` |
| `ShellNonZeroExit` | Not automatically an error — caller's `branch:` handles it |

---

### Section D: Multi-Tenant Isolation

#### Postgres Row-Level Security

Every table in the control plane DB has a `tenant_id UUID NOT NULL` column. RLS policies are enforced at the Postgres level (not the application level), so a compromised application cannot read another tenant's data even with a direct DB connection:

```sql
ALTER TABLE runs ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON runs
  USING (tenant_id = current_setting('app.current_tenant_id')::uuid);
```

The application layer sets `app.current_tenant_id` at the start of each request via a connection pool middleware that reads the tenant from the authenticated JWT claim.

#### Secret Management

`.agent` files must never contain plaintext secrets. API keys are referenced as `env("OPENAI_API_KEY")` expressions. At pipeline registration time, the user provides secret values via the web UI; these are encrypted with a per-tenant KMS key (AWS KMS, one key per tenant) and stored in the `tenant_secrets` table. At runtime, the secret manager decrypts secrets on demand and injects them as environment variables into tool handlers without logging them.

#### Execution Sandbox

`shell.run` executes in a per-tenant Docker container with the following constraints:

- **Network**: Isolated bridge network; no internet egress unless `http.get` is declared in the pipeline (which is allow-listed separately).
- **CPU**: 0.5 vCPU limit (cgroup v2).
- **Memory**: 512 MB limit.
- **Filesystem**: Read-only root; writable `/workspace` tmpfs only.
- **Execution time**: 60-second timeout; SIGKILL on expiry.
- **seccomp**: `default` Docker seccomp profile (blocks ~44 syscalls).
- **User**: Non-root `conductor` user (UID 1000).

Containers are pre-warmed per tenant (one standby container per tenant with a recent pipeline run) to reduce cold-start latency.

#### S3 Artifact Isolation

All `fs.*` tool operations resolve paths within a tenant-specific S3 prefix: `s3://conductor-artifacts/{tenant_id}/`. Path traversal attacks are blocked by the `FsHandler` which validates that the resolved path starts with the tenant prefix before issuing any S3 API call.

---

### Section E: GPU Serving Layer

#### vLLM Deployment Configuration

Tier 2 (Gemma4-26B-A4B) and Tier 3 (Gemma4-12B) are served via vLLM on AWS `g5.xlarge` instances (1× NVIDIA A10G, 24 GB VRAM). The serving command for Tier 2:

```bash
vllm serve google/gemma-4-26B-A4B-it \
  --speculative-model google/gemma-4-26B-A4B-it-assistant \
  --num-speculative-tokens 5 \
  --speculative-draft-tensor-parallel-size 1 \
  --quantization awq \
  --dtype bfloat16 \
  --max-model-len 32768 \
  --tensor-parallel-size 1 \
  --port 8000
```

AWQ (Activation-aware Weight Quantization) reduces model memory footprint while preserving quality within 1-2% on code tasks. `bfloat16` is preferred over `float16` for stability on the A10G's BF16 tensor cores.

#### MTP Speculative Decoding

**Multi-Token Prediction (MTP)** is a speculative decoding variant where a small drafter model proposes N tokens simultaneously, and the full target model verifies all N in a single forward pass. This yields 2-3× throughput improvement when the drafter's proposals are accepted at high rates (typically >70% for in-distribution coding tasks).

- **Drafter model**: `gemma-4-26B-A4B-it-assistant` (a fine-tuned assistant variant of the same checkpoint optimized for single-token prediction speed).
- **N = 5**: Each speculative step proposes 5 tokens; empirically optimal on coding tasks at this model size.
- **Adaptive scheduler**: vLLM's built-in adaptive speculative decoding dynamically adjusts N downward when acceptance rate drops (e.g., on unusual prompts), preventing throughput regression.
- **Target throughput**: ~60 tok/s for Tier 2 on a single A10G under normal load.

#### Gemma4-26B-A4B Architecture Fit

Gemma4-26B-A4B is a Mixture-of-Experts model with 26B total parameters but only 4B active parameters per forward pass (routing activates 4 of the expert FFN blocks per token). This means:

- **Memory**: VRAM consumption is proportional to 4B active params during inference, not 26B (weights are all loaded, but only active expert weights are processed in each forward pass).
- **Quality**: 26B knowledge capacity (all experts trained on the full dataset) — competitive with 20B+ dense models.
- **Speed**: Compute per token is proportional to 4B active params, enabling the ~60 tok/s target on a single A10G.
- **License**: Apache 2.0 — no restrictions on commercial API service.

#### Multi-Region Deployment

Three instances across AWS regions provide geographic latency reduction and redundancy:

| Region | Instance | Role |
|--------|----------|------|
| `us-east-1` | `g5.xlarge` spot | Primary (lowest latency for US East users) |
| `us-west-2` | `g5.xlarge` spot | Secondary (US West + failover) |
| `eu-west-1` | `g5.xlarge` spot | EU (GDPR-compliant data residency option) |

All three run as spot instances. Spot interruption handling: vLLM is stateless; in-flight requests are retried on the next available instance by the model router with a 5-second timeout for a graceful drain signal.

#### Health Check and Failover

The model router polls each vLLM endpoint at `/health` every 10 seconds. An instance is marked `UNHEALTHY` after 2 consecutive failures and removed from the routing pool. A new spot instance is launched automatically via an Auto Scaling Group (target capacity = 1 per region; the ASG replaces interrupted instances within 2 minutes). If fewer than 2 regions are healthy, the router falls back to Tier 1 (Bedrock) for all Tier 2 requests, with a user-visible warning.

---

### Section F: Billing Architecture

#### Credit System

1 credit = 1,000 Bedrock input tokens (or equivalent for output tokens, priced at 3× input rate). The credit system provides a unified billing abstraction across all providers and tiers.

Each tenant account has a `credit_balance` field in the `tenants` table. Credits are consumed at the time of `LLM_CALL` dispatch (pre-deduction) and reconciled against actual usage at the end of each billing period.

#### Stripe Metered Billing

After each `LLM_CALL` instruction completes, the billing engine posts a Stripe usage record:

```python
stripe.SubscriptionItem.create_usage_record(
    subscription_item_id=tenant.stripe_subscription_item_id,
    quantity=actual_tokens_used // 1000,  # in credits
    timestamp=int(time.time()),
    action="increment",
)
```

Usage records are posted asynchronously to a Redis queue and flushed to Stripe in batches every 60 seconds to avoid Stripe API rate limits. The Stripe metered billing webhook fires at the end of each billing period to finalize the invoice.

#### Bedrock Pass-Through

For Builder and Team tier users invoking Bedrock models (claude-*), billing flows as:

1. **Conductor bills the customer** via Stripe at Conductor's listed price per credit.
2. **Conductor is billed by AWS** for Bedrock tokens consumed. Monthly reconciliation compares Conductor's Stripe revenue from Bedrock usage against the AWS Bedrock invoice; the margin (target: 20%) is tracked in the billing dashboard.

The pass-through model requires that Conductor pre-purchase Bedrock capacity or operate on-demand. In Phase 1, on-demand only. In Phase 3, evaluate Bedrock provisioned throughput for large customers.

#### Budget Enforcement

Before dispatching any `LLM_CALL`, the runtime checks:

```python
estimated_cost = estimate_tokens(prompt) * COST_PER_TOKEN[model]
if tenant.credit_balance < estimated_cost:
    raise CreditExhaustedError(f"Insufficient credits: need {estimated_cost}, have {tenant.credit_balance}")
```

`estimate_tokens` uses a fast heuristic (chars/4) before the actual API call; the actual usage is reconciled after. If a tenant runs out of credits mid-pipeline, the run transitions to `FAILED` with a `CreditExhausted` reason, and all subsequent steps are skipped.

---

### Section G: MCP Integration

#### Conductor as MCP Server

The Conductor tool registry is exposed as an MCP (Model Context Protocol) server endpoint. Clients like Claude Desktop, Cursor, and Zed can connect to `https://api.conductor.dev/mcp` and discover all tools in the registry as MCP tool definitions.

Each Conductor tool (e.g., `fs.glob`, `shell.run`) is mapped to an MCP tool descriptor with JSON Schema input/output types. Authentication is via API key passed in the `Authorization: Bearer` header of the SSE connection.

#### Conductor as MCP Client

`.agent` files may reference external MCP servers as tool sources:

```agent
pipeline with_external_tools:
  mcp: https://mcp.example.com/tools

  step use_external:
    tool: example.search_docs(query="conductor DSL")
    output: $docs
```

The compiler fetches the MCP server's tool manifest at compile time and registers the discovered tools in the type checker's registry for that compilation unit. At runtime, the MCP client handler proxies `CALL` instructions for external tools to the MCP server via JSON-RPC.

#### Wire Protocol

MCP communication uses JSON-RPC 2.0 over HTTP with Server-Sent Events for streaming:

- **Tool discovery**: `GET /tools` → JSON array of tool descriptors
- **Tool invocation**: `POST /tools/{tool_name}` with JSON body `{params: {...}}` → JSON result
- **Streaming**: `Accept: text/event-stream` on tool invocation → SSE stream of partial results (buffered in v1; streamed to UI in Phase 2)

---

### Section H: Control Plane API

#### REST Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/compile` | Submit `.agent` source; returns IR or compiler errors |
| `POST` | `/runs` | Submit compiled IR for execution; returns `run_id` |
| `GET` | `/runs/{run_id}` | Get run status and current state |
| `GET` | `/runs/{run_id}/events` | SSE stream of live execution events |
| `POST` | `/runs/{run_id}/approve` | Submit human checkpoint approval |
| `POST` | `/runs/{run_id}/reject` | Submit human checkpoint rejection |
| `POST` | `/runs/{run_id}/abort` | Cancel a running pipeline |
| `GET` | `/pipelines` | List tenant's saved pipelines |
| `POST` | `/pipelines` | Save a named pipeline |

#### WebSocket / SSE for Live Events

`GET /runs/{run_id}/events` returns an SSE stream. Event types:

| Event | Payload |
|-------|--------|
| `ir_start` | `{pc, instruction}` |
| `ir_complete` | `{pc, instruction, result, duration_ms}` |
| `llm_token` | `{pc, token}` (streaming, Phase 2) |
| `checkpoint` | `{pc, display_data}` |
| `state_change` | `{from, to}` |
| `error` | `{code, message, pc}` |
| `done` | `{total_instructions, total_tokens, duration_ms}` |

#### Authentication

All API endpoints require a JWT in the `Authorization: Bearer` header. JWTs are issued by Clerk (Phase 1) or Auth0 (Phase 3 Enterprise). The JWT contains `sub` (user ID), `tenant_id`, and `tier` claims. The control plane API validates the JWT and sets `app.current_tenant_id` on the Postgres connection before any query.

---

## Alternatives Considered

### 1. Why Not Build on LangGraph?

LangGraph is a Python library for building stateful multi-agent workflows as directed graphs. Building Conductor on top of LangGraph would mean:

- **Provider lock-in via Python typing**: LangGraph nodes are Python callables; switching LLM providers requires changing Python imports and function signatures across every node.
- **No compile-time type safety**: Tool contracts are Python function signatures enforced only at runtime, with no static analysis path.
- **Python coupling is the product**: Conductor's value proposition is being a language-agnostic, provider-agnostic artifact. A LangGraph-based Conductor would inherit Python's import-time overhead and would be unusable from non-Python runtimes.
- **No auditability**: LangGraph's state graph is a Python object; it cannot be serialized to a human-readable IR without significant additional engineering.

**Decision**: Custom language and runtime. We accept the engineering cost of building the compiler pipeline in exchange for complete control over the abstraction.

### 2. Why Not a Visual Workflow Builder (n8n-style)?

Visual builders (n8n, Zapier, Flowise) are excellent for non-technical users but are not code-native:

- **Not git-versionable**: Visual workflow state is stored in a DB export (JSON blob), not as diffable source code.
- **Not composable**: Visual nodes cannot be imported or parameterized in the way that `.agent` files can.
- **Target user mismatch**: Conductor's target is senior developers who already write code; a visual builder would attract a different (and larger, but less monetizable) audience.

**Decision**: Code-first, text-based DSL. The `.agent` file is the source of truth and lives in version control.

### 3. Why Python for the Compiler, Not Go/Rust?

Python advantages for Phase 1:
- Faster iteration (no compile step; changes visible immediately).
- Rich ecosystem: `pydantic` for IR type validation, `pytest` for compiler test suite, `tiktoken` for token counting.
- Team familiarity: the founding team writes Python fluently.

Go/Rust advantages (deferred to Phase 3):
- Compiler throughput: a Go compiler service could handle 10× the requests per second on the same hardware.
- Binary distribution: a Rust CLI compiler could be distributed as a single binary for local use without a Python runtime.

**Decision**: Python for Phase 1–2. Profile compiler latency in Phase 2; if p99 > 500ms on real user workloads, rewrite the hot path (lexer + parser) in Go as a compiled extension.

### 4. Why Recursive Descent vs ANTLR/PLY?

ANTLR and PLY generate parsers from a grammar file, which would be ideal if the grammar were stable. In Phase 1, the grammar is expected to evolve rapidly based on user feedback (from the syntax test experiment). A generated parser requires rerunning the generator on every grammar change; a hand-written recursive descent parser changes where the grammar changes.

Additionally:
- No external build dependency (ANTLR requires a JVM; PLY requires PLY).
- Easier error recovery: hand-written parsers can implement domain-specific recovery heuristics.
- Easier to maintain: the parser code is readable Python, not a generated file.

**Decision**: Hand-written recursive descent. Revisit if grammar stabilizes and performance becomes a concern.

### 5. Why Gemma4 vs Llama 3?

| Criterion | Gemma4-26B-A4B | Llama 3.3 70B |
|-----------|---------------|---------------|
| Active params / token | 4B | 70B |
| VRAM on A10G (AWQ) | ~18 GB | Does not fit on one A10G |
| MTP support | Yes (via Gemma MTP checkpoint) | No native MTP support in vLLM |
| License | Apache 2.0 | Llama 3 community license (usage restrictions at scale) |
| Code quality | Competitive on HumanEval/MBPP | Strong, but requires 70B |

Llama 3.3 70B cannot serve from a single A10G at the throughput required for the free tier's cost model. The MoE architecture of Gemma4 and its native MTP support are the deciding factors.

**Decision**: Gemma4-26B-A4B for Tier 2; Gemma4-12B for Tier 3. Re-evaluate if Llama 4 MoE models with MTP support become available.

---

## Open Questions

### OQ-1: Should .agent Files Be YAML-Based Instead of Custom Syntax?

**Argument for YAML**: YAML is already parsed by every major language. Developers know it. No lexer/parser to build.

**Argument against YAML**: YAML's indentation rules conflict with control flow constructs. `branch:` arms with `on:` conditions containing `->` arrows would require excessive quoting. Loop bodies with nested contexts are deeply indented and hard to read. YAML also has surprising parsing edge cases (the Norway problem, boolean auto-coercion) that would create subtle bugs in pipeline definitions.

**Proposed resolution**: Custom syntax in v1. After 3 months of user feedback from the beta, if ≥40% of users request YAML, build a YAML→.agent transpiler as a secondary input format rather than changing the primary syntax.

### OQ-2: How Should Streaming LLM Responses Integrate With the IR?

Currently, `LLM_CALL` buffers the complete response before the runtime advances to the next instruction. This means the user sees no output until the full LLM response is complete.

**Alternative**: Pass the streaming token iterator through to the web UI while the runtime suspends at the `LLM_CALL` instruction until the stream completes.

**Proposed resolution**: Buffer in v1 (simpler; avoids partial JSON parsing of structured outputs). Add streaming-passthrough in Phase 2 using the `llm_token` SSE event type already defined in the control plane API. The IR instruction set does not change; only the runtime's buffering behavior changes.

### OQ-3: Should the Type System Support Union Types for Tool Outputs?

`human.checkpoint` returns `{status: "approved" | "rejected", notes: String}`. The `status` field is a string union type. Currently the type checker treats this as `String` and does not validate that branch conditions cover both arms.

**Proposed resolution**: No union types in v1. The type checker emits a `SYM002`-equivalent warning if a branch on a known-union field does not include a `default` arm. Add proper union type inference in Phase 2 when we have enough user data to know which type errors matter most.

---

## Security Considerations

**shell.run sandbox**: The Docker execution model (per-tenant container, seccomp, non-root user, network isolation, CPU/memory limits) is the primary security boundary for arbitrary command execution. The sandbox must be validated by penetration testing before Phase 1 beta launch (see go/no-go criteria).

**Secret injection**: Secrets are decrypted per-request by the secret manager and injected as environment variables into tool handler processes. They are never logged, never included in IR artifacts, and never transmitted to LLM providers (unless the user explicitly includes them in a prompt template, which is a user error the type checker will warn about via a `SecretInPrompt` heuristic).

**Prompt injection**: Retrieved content (`retrieve:` sources) is passed verbatim to LLM calls. A malicious document could attempt to override the user's prompt instructions. Mitigation in v1: prepend a system instruction marker distinguishing user instructions from retrieved content. Full mitigation requires adversarial robustness work deferred to Phase 3.

**Dynamic tool args**: If tool arguments are constructed from LLM output (e.g., `tool: fs.write(path=$llm_output.file)`), the LLM output could inject path traversal sequences. Mitigation: the `FsHandler` validates the resolved path against the tenant S3 prefix before every operation (see Section D).

---

## Success Criteria

This RFC is "done" when:

1. The canonical `refactor_session.agent` compiles with zero type errors and 28 IR instructions in under 200ms on a developer laptop.
2. The runtime executes all 28 instructions end-to-end, producing correct file writes and a passing `npm test` exit code.
3. The human checkpoint pauses execution and resumes correctly after approval via the web UI.
4. A second tenant's run cannot read the first tenant's artifacts or variable state (verified by the security test suite).
5. Stripe usage records within the run match actual Bedrock token counts within 5%.

---

## References

- [Compiler Theory Foundation](/wiki/index.md)
- [Runtime Architecture North Star](/harness/README.md) — reliability math, harness engineering patterns, capability acquisition ladder
- [Ch3 Lexical Analysis](/wiki/chapters/ch03-lexical-analysis/index.md)
- [Ch4 Parsing](/wiki/chapters/ch04-parsing/index.md)
- [Ch5 AST](/wiki/chapters/ch05-ast/index.md)
- [Ch6 Symbol Table](/wiki/chapters/ch06-symbol-table/index.md)
- [Ch7 Type Checking](/wiki/chapters/ch07-type-checking/index.md)
- [Ch8 IR / Bytecode](/wiki/chapters/ch08-bytecode/index.md)
- [Ch9 VM / Runtime](/wiki/chapters/ch09-bytecode-vm/index.md)
- [Go/No-Go Criteria](/startup/04-validation/go-no-go-criteria.md)
- [Validation Experiments](/startup/04-validation/experiments.md)
