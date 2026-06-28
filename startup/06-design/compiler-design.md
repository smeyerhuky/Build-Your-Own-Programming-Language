---
type: Design Doc
title: "Conductor Compiler Design"
description: Multi-phase compiler pipeline for the Conductor .agent DSL, covering lexer, parser, AST node hierarchy, task graph builder, symbol table, type checker, and IR emitter. Maps directly to chapters 3–8 of Build Your Own Programming Language (Jeffery, Packt).
resource: /startup/06-design/compiler-design.md
tags: [compiler, lexer, parser, ast, ir, type-checker, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor Compiler Design

## 1. Overview

The Conductor compiler translates `.agent` source files into a JSON IR instruction list that the runtime orchestrator can walk step-by-step. It follows the classical pipeline from *Build Your Own Programming Language* (Jeffery, Packt), with each stage implemented as a separate Python module with a clean, well-typed interface. The compiler is **stateless and synchronous** — given source bytes in, IR JSON out — which means it is trivially horizontally scalable and has no shared state between requests.

The compiler pipeline has seven phases:

```
Source bytes
    │
    ▼
┌─────────┐    Token stream
│  Lexer  │ ──────────────────────────────────────► (Ch3)
└─────────┘
    │
    ▼
┌──────────┐   PipelineNode (AST root)
│  Parser  │ ──────────────────────────────────────► (Ch4)
└──────────┘
    │
    ▼
┌─────────────┐   Directed graph of StepNodes
│ Task Graph  │ ──────────────────────────────────► (Ch5)
│   Builder   │
└─────────────┘
    │
    ▼
┌──────────────┐   Scope chain, $var → (StepNode, Type)
│ Symbol Table │ ──────────────────────────────────► (Ch6)
└──────────────┘
    │
    ▼
┌──────────────┐   Type-annotated Task Graph
│ Type Checker │ ──────────────────────────────────► (Ch7)
└──────────────┘
    │
    ▼
┌────────────┐   list[IRInstruction]
│ IR Emitter │ ──────────────────────────────────► (Ch8)
└────────────┘
    │
    ▼
IR JSON  (consumed by runtime orchestrator)
```

All phases share a `CompileContext` object that accumulates errors and warnings. Compilation proceeds through all phases even after errors in an earlier phase, collecting as many diagnostics as possible (up to a configured maximum). The final output is either `{ "ir": [...] }` on full success, or `{ "errors": [...], "warnings": [...] }` when any error-severity diagnostics exist.

---

## 2. Module Structure

```
conductor/
  compiler/
    __init__.py          # compile(source: str) -> CompileResult
    lexer.py             # Ch3: regex-based tokenizer, INDENT/DEDENT handling
    parser.py            # Ch4: recursive descent parser, error recovery
    ast_nodes.py         # Ch5: dataclass AST node hierarchy
    task_graph.py        # Ch5: Task Graph builder (AST → directed graph)
    symbol_table.py      # Ch6: scope chain, $variable resolution
    type_checker.py      # Ch7: tool signature registry, type inference
    ir_emitter.py        # Ch8: Task Graph → IR instruction list
    ir_types.py          # IRInstruction dataclass + all op type literals
    errors.py            # CompileError dataclass, error codes E001–E015
    types.py             # Type system: FileList, ExecResult, LLMOutput, etc.
```

The public entry point is `compile(source: str) -> CompileResult` in `__init__.py`. It instantiates each phase in sequence, passes outputs forward, and returns a `CompileResult(ir, errors, warnings)`.

---

## 3. Lexer (Ch3 Analog)

The lexer is a single-pass, regex-based tokenizer implemented in pure Python using the `re` module. It handles significant whitespace (indentation-sensitive syntax, like Python) using an explicit indent stack.

**Token Dataclass**
```python
from dataclasses import dataclass
from enum import auto, Enum

class TokenType(Enum):
    # Structural
    INDENT = auto()
    DEDENT = auto()
    NEWLINE = auto()
    EOF = auto()
    # Keywords
    PIPELINE = auto()
    STEP = auto()
    MODEL = auto()
    CONTEXT = auto()
    PROMPT = auto()
    TOOL = auto()
    OUTPUT = auto()
    BRANCH = auto()
    CONDITION = auto()
    LOOP = auto()
    ON = auto()
    FOR = auto()
    IN = auto()
    DEFAULT = auto()
    BUDGET = auto()
    RETRIEVE = auto()
    # Literals and identifiers
    STRING = auto()       # "..." or triple-quoted
    INTEGER = auto()
    IDENTIFIER = auto()
    DOLLAR_VAR = auto()   # $var_name
    ARROW = auto()        # ->
    COLON = auto()
    COMMA = auto()
    LPAREN = auto()
    RPAREN = auto()
    LBRACKET = auto()
    RBRACKET = auto()
    DOT = auto()
    EQ = auto()           # ==
    NEQ = auto()          # !=
    TEMPLATE_START = auto()  # {{
    TEMPLATE_END = auto()    # }}

@dataclass(frozen=True)
class Token:
    type: TokenType
    lexeme: str
    line: int
    col: int
```

**Indent Handling**
The lexer maintains an `indent_stack: list[int]` initialized to `[0]`. On each non-blank, non-comment line:
1. Count leading spaces. Tabs are rejected (LexError E000).
2. If indent > stack top: push new level, emit one `INDENT` token.
3. If indent < stack top: pop until stack top matches indent; emit one `DEDENT` per pop. If no level matches, emit LexError E000 (inconsistent dedent).
4. If indent == stack top: no structural token.

**Error**
`LexError(code: str, message: str, line: int, col: int)` is accumulated into `CompileContext.errors` and the lexer continues scanning (error recovery: skip to next whitespace).

---

## 4. Parser (Ch4 Analog)

The parser is a hand-written recursive descent parser with no external grammar library dependencies. Each non-terminal in the grammar corresponds to a method on the `Parser` class.

**Key Grammar Methods**

```python
class Parser:
    def parse_program(self) -> PipelineNode:
        """Entry point. Expects exactly one pipeline declaration."""

    def parse_pipeline(self) -> PipelineNode:
        """pipeline <Name>: INDENT [model_decl]* [context_decl]? step+ DEDENT"""

    def parse_model_decl(self) -> tuple[str, str]:
        """model <alias>: <model_name>  →  (alias, model_name)"""

    def parse_context_decl(self) -> int:
        """context budget: <N> tokens  →  N (integer token count)"""

    def parse_step(self) -> StepNode:
        """step <Name>: INDENT [step_field]+ DEDENT"""

    def parse_step_field(self, step: StepNode) -> None:
        """Dispatch to: model/context/prompt/tool/output/branch/condition/loop"""

    def parse_context_sources(self) -> list[ContextSource]:
        """[source, source, ...]  where source is $var | retrieve(...) | fs.read(...)"""

    def parse_branch(self) -> BranchNode:
        """branch: INDENT (- on: <cond> -> <step>)+ DEDENT"""

    def parse_condition(self) -> ConditionNode:
        """condition: INDENT (- on: <expr> -> <step>)+ DEDENT"""

    def parse_loop(self) -> LoopNode:
        """loop: for $item in $list INDENT [loop_field]+ DEDENT"""

    def parse_tool_call(self) -> ToolCallNode:
        """tool: <name>.<method>(<args>)"""

    def parse_prompt_string(self) -> str:
        """Handles regular strings and template strings with {{ $var }} interpolation"""
```

**Error Recovery**
On a parse error within a step, the parser emits the error and advances past tokens until it finds a `STEP` keyword at the top-level indent (or `PIPELINE` / `EOF`). This allows the parser to report errors in multiple steps in a single compile pass, matching the diagnostic experience of mature compilers.

---

## 5. AST Node Hierarchy (Ch5 Analog)

All AST nodes inherit from `ASTNode` and are frozen dataclasses (immutable after construction).

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import Optional

@dataclass(frozen=True)
class ASTNode:
    line: int
    col: int

@dataclass(frozen=True)
class PipelineNode(ASTNode):
    name: str
    model_aliases: dict[str, str]   # e.g. {"default": "claude-3-5-sonnet"}
    budget_tokens: Optional[int]    # from "context budget: 32k tokens"
    steps: list[StepNode]

@dataclass(frozen=True)
class StepNode(ASTNode):
    name: str
    model_alias: Optional[str]       # references PipelineNode.model_aliases key
    context_sources: list[ContextSource]
    prompt: Optional[str]            # may contain {{ $var }} templates
    tool_call: Optional[ToolCallNode]
    output_var: Optional[str]        # the $var name, without leading $
    branch: Optional[BranchNode]
    condition: Optional[ConditionNode]
    loop: Optional[LoopNode]

@dataclass(frozen=True)
class ContextSource:
    """One item in a context: [...] declaration."""
    kind: str                        # "var" | "retrieve" | "fs_read" | "inline"
    var_name: Optional[str]          # for kind="var": the $var name
    path: Optional[str]              # for kind="retrieve" | "fs_read"
    inline: Optional[str]            # for kind="inline"

@dataclass(frozen=True)
class ToolCallNode(ASTNode):
    namespace: str                   # e.g. "fs", "shell", "human"
    method: str                      # e.g. "glob", "run", "checkpoint"
    args: list[ToolArg]

@dataclass(frozen=True)
class ToolArg:
    """Positional or keyword argument to a tool call."""
    kind: str                        # "string" | "var" | "integer" | "keyword"
    value: str                       # raw string value or $var name
    keyword: Optional[str]           # for kind="keyword": the arg name

@dataclass(frozen=True)
class BranchNode(ASTNode):
    """branch: - on: <decision_value> -> <step_name>"""
    input_var: str                   # the $var holding the decision
    arms: list[BranchArm]

@dataclass(frozen=True)
class BranchArm:
    on_value: str                    # e.g. "approved", "rejected"
    target_step: str                 # step name to jump to

@dataclass(frozen=True)
class ConditionNode(ASTNode):
    """condition: - on: <expr> -> <step_name>"""
    arms: list[ConditionArm]

@dataclass(frozen=True)
class ConditionArm:
    expr: str                        # e.g. "exit_code == 0"
    target_step: str

@dataclass(frozen=True)
class LoopNode(ASTNode):
    """loop: for $item in $list"""
    item_var: str                    # e.g. "item"
    list_var: str                    # e.g. "dead_items"
    body_context: list[ContextSource]
    body_prompt: Optional[str]
    body_output_var: Optional[str]
    body_tool_calls: list[ToolCallNode]
```

---

## 6. Task Graph Builder (Ch5 Analog)

The task graph builder converts the parsed `PipelineNode` into a directed graph where each node is a `StepNode` and edges represent either **data dependencies** (one step's output feeds another's context) or **control flow** (branch/condition arms).

```python
from dataclasses import dataclass, field
from enum import auto, Enum

class EdgeKind(Enum):
    DATA = auto()       # $var flows from source to target
    CONTROL = auto()    # branch/condition arm

@dataclass
class TaskEdge:
    source: str         # step name
    target: str         # step name
    kind: EdgeKind
    label: Optional[str]  # for CONTROL edges: the condition value

@dataclass
class TaskGraph:
    steps: dict[str, StepNode]      # name → StepNode
    edges: list[TaskEdge]
    back_edges: list[TaskEdge]      # detected by DFS; represent loops
    entry: str                      # first step name
```

**Algorithm**
1. Register all steps in `steps` dict. Detect duplicate step names (error E003).
2. For each step, scan `context_sources`, `prompt` template vars, `loop.list_var`, `branch.input_var`:
   - If a `$var` is referenced, find the step that declares it as `output_var` → add DATA edge.
3. For each branch/condition arm:
   - Add CONTROL edge (source=current step, target=arm's `target_step`, label=arm value).
4. Run DFS from entry step, coloring nodes WHITE→GRAY→BLACK:
   - GRAY→GRAY edge = back-edge (loop). Accumulate in `back_edges`, emit warning E007.
   - Unvisited steps = dead steps, emit warning E006.

**Back-Edge Policy**
Back-edges are **warnings, not errors**. They represent intentional loops (e.g., `Validate → FindDeadCode`). The IR emitter converts them into `JUMP` instructions targeting a `LABEL` emitted before the target step. A maximum iteration limit is enforced at runtime (1000 iterations per LOOP_START or JUMP-back), not at compile time.

---

## 7. Symbol Table (Ch6 Analog)

The symbol table resolves `$var` references to the step that produces them and the inferred type.

```python
@dataclass
class Symbol:
    var_name: str
    defined_by: StepNode           # the step with output_var == var_name
    type: Type                     # inferred by type checker later
    scope: "Scope"

class Scope:
    def __init__(self, parent: Optional["Scope"] = None):
        self.parent = parent
        self.symbols: dict[str, Symbol] = {}

    def define(self, name: str, symbol: Symbol) -> None:
        if name in self.symbols:
            raise DuplicateDefinitionError(name)
        self.symbols[name] = symbol

    def resolve(self, name: str) -> Optional[Symbol]:
        if name in self.symbols:
            return self.symbols[name]
        if self.parent:
            return self.parent.resolve(name)
        return None
```

**Scope Chain**
- `PipelineScope`: top-level. All steps' `output_var` declarations go here.
- `LoopScope(parent=PipelineScope)`: created for each `LoopNode`. The loop's `item_var` is defined here, shadowing any same-named pipeline-level variable (this emits warning E008 if shadowing occurs).

**Resolution**
`resolve($var, current_scope)` walks the chain from innermost scope outward. If no symbol is found, error E001 ("undefined variable `$var` in step `StepName`") is emitted.

---

## 8. Type Checker (Ch7 Analog)

The type checker walks the task graph in **topological order** (using the back-edge-free subgraph), assigns types to all `$var` symbols, and validates all tool call signatures and field accesses.

**Type System**

```python
from typing import Union

class Type:
    pass

@dataclass
class FileListType(Type):
    """Result of fs.glob() — list of file paths."""

@dataclass
class FileContentType(Type):
    """Result of fs.read() — string content."""

@dataclass
class ExecResultType(Type):
    """Result of shell.run() — {exit_code: int, stdout: str, stderr: str}."""
    fields = {"exit_code": IntType(), "stdout": StringType(), "stderr": StringType()}

@dataclass
class LLMOutputType(Type):
    """Result of an LLM_CALL step — unstructured string unless prompt specifies JSON."""

@dataclass
class JSONListType(Type):
    """LLM output parsed as JSON array. item_type is inferred from prompt annotation."""
    item_fields: dict[str, Type]   # e.g. {"file": StringType, "line": IntType}

@dataclass
class CheckpointDecisionType(Type):
    """Result of human.checkpoint() — {decision: str}."""

@dataclass
class StringType(Type): pass
@dataclass
class IntType(Type): pass
@dataclass
class AnyType(Type): pass    # accepted by context_sources (any type is valid)
```

**Tool Signature Registry**

```python
@dataclass
class ArgSpec:
    name: str
    type: Type
    required: bool = True

@dataclass
class ToolSig:
    name: str                  # e.g. "fs.glob"
    args: list[ArgSpec]
    return_type: Type

TOOL_REGISTRY: dict[str, ToolSig] = {
    "fs.glob":          ToolSig("fs.glob",    [ArgSpec("pattern", StringType())], FileListType()),
    "fs.read":          ToolSig("fs.read",    [ArgSpec("path",    StringType())], FileContentType()),
    "fs.write":         ToolSig("fs.write",   [ArgSpec("path",    StringType()),
                                               ArgSpec("content", AnyType())],   StringType()),
    "shell.run":        ToolSig("shell.run",  [ArgSpec("command", StringType())], ExecResultType()),
    "human.checkpoint": ToolSig("human.checkpoint", [ArgSpec("data", AnyType())], CheckpointDecisionType()),
    "http.get":         ToolSig("http.get",   [ArgSpec("url",     StringType())], StringType()),
    "http.post":        ToolSig("http.post",  [ArgSpec("url",     StringType()),
                                               ArgSpec("body",    AnyType())],   StringType()),
}
```

**Validation Rules**
1. `context_sources`: any type accepted (AnyType constraint satisfied by all types).
2. Branch `input_var`: must resolve; `on` values must be string literals (no type constraint on branch target var).
3. Condition `expr` field access (e.g., `exit_code`): the `$var` must have a type with a `fields` dict, and the field must be present. Error E002 if not.
4. Loop `list_var`: must be `FileListType` or `JSONListType`. The `item_var` gets type `FileContentType` (for FileList) or the list's `item_fields` type (for JSONList). Error E004 if loop over non-list type.
5. Tool call args: each arg is checked against the `ToolSig.args` spec. Unknown tool → error E005.
6. Model alias: each `model:` declaration in a step must resolve to a key in `PipelineNode.model_aliases`. Error E009 if unknown alias.

---

## 9. IR Emitter (Ch8 Analog)

The IR emitter performs a **topological sort** of the task graph (excluding back-edges) and emits one or more IR instructions per step.

**IRInstruction Dataclass**

```python
@dataclass
class IRInstruction:
    op: str                        # operation name (see IR instruction set below)
    args: dict[str, Any]           # operation-specific arguments
    step_name: Optional[str]       # source step for debugging
    line: Optional[int]            # source line for error messages
```

**Full IR Instruction Set**

| Op | Args | Description |
|---|---|---|
| `LLM_CALL` | `model_alias, prompt_template, context_refs[], output_var` | Call the LLM; assign response to output_var |
| `CALL` | `tool_name, args[], output_var` | Invoke a tool handler |
| `ASSIGN` | `var, value` | Assign a literal or computed value to a variable |
| `BRANCH` | `input_var, field, cases[]` | Read field from input_var; jump to matching label |
| `LOOP_START` | `list_var, item_var, loop_label` | Initialize iterator; set up loop scope |
| `LOOP_END` | `loop_label` | Advance iterator; jump back to LOOP_START or exit |
| `CONTEXT_PUSH` | `sources[]` | Resolve sources; push entry onto context stack |
| `CONTEXT_POP` | — | Pop top entry from context stack |
| `RETRIEVE` | `path, output_var` | Load file from bundle; cache; assign |
| `CHECKPOINT` | `display_data_var, output_var` | Serialize data; emit SSE; pause run |
| `SUMMARY` | `template, output_var` | Summarize top context entry; assign |
| `JUMP` | `label` | Unconditional jump (back-edges, forced Abort) |
| `LABEL` | `name` | Jump target marker |
| `HALT` | `message_var` | Terminate run with message |

**Emit Algorithm**

```python
def emit(self, graph: TaskGraph) -> list[IRInstruction]:
    ir: list[IRInstruction] = []
    visited: set[str] = set()

    # Emit a LABEL before every step that is a target of a back-edge or branch
    jump_targets = {e.target for e in graph.edges + graph.back_edges
                    if e.kind == EdgeKind.CONTROL}

    for step_name in self.topological_order(graph):
        step = graph.steps[step_name]

        if step_name in jump_targets:
            ir.append(IRInstruction(op="LABEL", args={"name": step_name},
                                    step_name=step_name, line=step.line))

        if step.context_sources:
            ir.append(IRInstruction(op="CONTEXT_PUSH",
                                    args={"sources": self.serialize_sources(step.context_sources)},
                                    step_name=step_name, line=step.line))

        if step.tool_call:
            tool_name = f"{step.tool_call.namespace}.{step.tool_call.method}"
            if tool_name == "human.checkpoint":
                ir.append(IRInstruction(op="CHECKPOINT",
                                        args={"display_data_var": step.tool_call.args[0].value,
                                              "output_var": step.output_var},
                                        step_name=step_name, line=step.line))
            else:
                ir.append(IRInstruction(op="CALL",
                                        args={"tool_name": tool_name,
                                              "args": [a.value for a in step.tool_call.args],
                                              "output_var": step.output_var},
                                        step_name=step_name, line=step.line))

        elif step.prompt:
            ir.append(IRInstruction(op="LLM_CALL",
                                    args={"model_alias": step.model_alias or "default",
                                          "prompt_template": step.prompt,
                                          "context_refs": [],  # resolved by runtime from context stack
                                          "output_var": step.output_var},
                                    step_name=step_name, line=step.line))

        if step.context_sources:
            ir.append(IRInstruction(op="CONTEXT_POP", args={},
                                    step_name=step_name, line=step.line))

        if step.loop:
            ir.extend(self.emit_loop(step, graph))

        if step.branch:
            ir.extend(self.emit_branch(step))

        if step.condition:
            ir.extend(self.emit_condition(step))

    # Back-edges become JUMPs at the point after the source step's last instruction
    for edge in graph.back_edges:
        ir.append(IRInstruction(op="JUMP",
                                args={"label": edge.target},
                                step_name=edge.source, line=None))

    ir.append(IRInstruction(op="HALT", args={"message_var": None},
                            step_name=None, line=None))
    return ir
```

---

## 10. Error Reporting Format

The compiler returns a unified JSON error report:

```json
{
  "errors": [
    {
      "code": "E002",
      "message": "Field 'exit_code' does not exist on type 'LLMOutput'. Expected ExecResult.",
      "line": 42,
      "column": 14,
      "step": "Validate",
      "severity": "error"
    },
    {
      "code": "E001",
      "message": "Undefined variable '$patched' referenced in step 'Validate'.",
      "line": 51,
      "column": 20,
      "step": "Validate",
      "severity": "error"
    }
  ],
  "warnings": [
    {
      "code": "E007",
      "message": "Back-edge detected: Validate → FindDeadCode. This creates a potential infinite loop; max iterations enforced at runtime.",
      "step": "Validate",
      "severity": "warning"
    }
  ]
}
```

**Error Code Registry**

| Code | Phase | Description |
|---|---|---|
| E000 | Lexer | Invalid token or inconsistent indentation |
| E001 | Symbol Table | Undefined variable |
| E002 | Type Checker | Field access on incompatible type |
| E003 | Task Graph | Duplicate step name |
| E004 | Type Checker | Loop over non-list type |
| E005 | Type Checker | Unknown tool name |
| E006 | Task Graph | Dead step (unreachable from entry) — warning |
| E007 | Task Graph | Back-edge detected (potential infinite loop) — warning |
| E008 | Symbol Table | Loop variable shadows pipeline-level variable — warning |
| E009 | Type Checker | Unknown model alias |
| E010 | IR Emitter | Step has neither prompt nor tool_call |
| E011 | Parser | Unexpected token |
| E012 | Parser | Missing required field in step |
| E013 | Type Checker | Branch target step does not exist |
| E014 | Type Checker | Condition expression references undefined variable |
| E015 | IR Emitter | Context budget value not parseable (e.g., "32k" → 32768) |

---

## 11. Compiler Performance Targets

| Metric | Target | Notes |
|---|---|---|
| p50 compile latency | <100ms | Typical pipeline (10 steps, 5 context refs) |
| p99 compile latency | <500ms | Large pipeline (50 steps, many branches) |
| Memory per request | <50MB | Stateless; no shared state between requests |
| Max source size | 1MB | Larger sources rejected at API gateway |
| Max steps per pipeline | 200 | Hard limit; error E015 if exceeded |
| Error recovery | Continue after each error | Report all errors in one pass |

The lexer and parser are the hottest phases. The type checker's topological sort is O(V + E) on the task graph. For a 50-step pipeline with an average of 3 edges per step, this is negligible. The dominant cost is tokenization of large `retrieve()` content bundles, which are loaded from S3 — but those are loaded at runtime, not compile time.
