---
type: Spec
title: "Conductor Language Specification v0.1"
description: Formal specification for the Conductor .agent DSL — lexical structure, complete EBNF grammar, type system with inference rules and error codes, semantic validity rules, and template interpolation semantics.
resource: /startup/05-specs/language-spec.md
tags: [language, dsl, ebnf, grammar, types, spec, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor Language Specification v0.1

**Status**: Draft  
**Applies to**: Conductor compiler v0.1.x  
**Conformance suite**: `tests/conformance/language/`

---

## 1. Overview

A Conductor program is a single `.agent` file that declares one **pipeline** — a named, typed workflow that compiles to a sequence of IR instructions dispatched to LLM APIs and tool executors. The pipeline is the top-level unit of compilation and deployment.

**File extension**: `.agent`  
**Encoding**: UTF-8. Files must be valid UTF-8; the compiler rejects files containing invalid byte sequences before lexing begins.  
**Line endings**: Both LF (`\n`) and CRLF (`\r\n`) are accepted. The lexer normalizes all line endings to LF before tokenizing. A file may not mix line-ending styles within a single line.  
**Maximum file size**: 1 MiB. Files exceeding this limit are rejected with a pre-lexer error.

---

## 2. Lexical Structure

### 2.1 Character Set

The source character set is Unicode (any code point valid in UTF-8). Unicode characters are permitted inside string literals, template expressions, comments, and identifier names. ASCII-only restriction applies only to keyword tokens and operator tokens.

### 2.2 Comments

**Line comments** begin with `#` and extend to (but do not include) the next LF. A `#` inside a string literal is not a comment delimiter.

```
# This is a comment
pipeline MyPipeline:  # inline comment
```

**Block comments** are not supported in v0.1. A future version may introduce `/* ... */` syntax; parsers must treat `/*` as a lexical error in v0.1.

### 2.3 Whitespace and Indentation

Spaces and tabs outside string literals are whitespace tokens. The lexer uses indentation to emit synthetic `INDENT` and `DEDENT` tokens using the following algorithm (modeled on Python's tokenizer):

1. Maintain a stack of indentation levels, initialized to `[0]`.
2. At the start of each logical line (after comment and blank-line stripping), count the leading spaces.
3. **Tabs are rejected**: if a tab character appears as leading whitespace, the lexer emits error `E-INDENT` and halts.
4. If the new indent level is greater than the top of the stack, push it and emit `INDENT`.
5. If the new indent level equals the top, emit nothing (same level).
6. If the new indent level is less than the top, pop levels (emitting `DEDENT` for each) until the top equals the new level. If no matching level is found, emit `E-INDENT`.
7. **Indent size**: each level must be exactly 2 spaces deeper than the enclosing level. A difference of 1, 3, or 4 spaces at an indent boundary is a lexical error.

Trailing whitespace on any line is silently discarded.

Blank lines (lines containing only whitespace or comments) do not contribute to indentation processing and do not generate `NEWLINE` tokens.

### 2.4 Token Types

The following table is the complete token vocabulary. The lexer matches tokens in priority order (keywords before identifiers, longer matches before shorter).

| Token | Regex / Description | Examples |
|---|---|---|
| `KEYWORD` | One of the reserved words listed below, not followed by `[A-Za-z0-9_]` | `pipeline`, `step`, `model`, `context`, `tool`, `prompt`, `output`, `branch`, `loop`, `condition`, `on`, `retrieve`, `summary`, `budget`, `for`, `in`, `tokens`, `default` |
| `IDENT` | `[A-Za-z][A-Za-z0-9_]*` (not matching a keyword) | `RefactorDeadCode`, `FindDeadCode`, `Ingest`, `codegen` |
| `DOLLAR_IDENT` | `[$][A-Za-z][A-Za-z0-9_]*` | `$source_files`, `$dead_items`, `$item` |
| `STRING` | `"([^"\\]|\\.)*"` or `'([^'\\]|\\.)*'` — double or single quoted, with standard C escape sequences (`\n`, `\t`, `\\`, `\"`, `\'`, `\uXXXX`) | `"claude-3-5-sonnet"`, `"./src/**/*.ts"`, `'npm test'` |
| `NUMBER` | `[0-9]+([kK])?` — an integer literal optionally suffixed with `k` or `K` (multiplied by 1024 at parse time) | `32k` → 32768, `5000`, `32768`, `0` |
| `TEMPLATE` | `\{\{[^}]*\}\}` — matched as a single token within STRING context during prompt/summary parsing (see §6) | `{{ $item.line }}`, `{{ len($dead_items) }}` |
| `ARROW` | `->` | `-> RefactorEach` |
| `COLON` | `:` | `model default:` |
| `LBRACKET` | `[` | context list open |
| `RBRACKET` | `]` | context list close |
| `COMMA` | `,` | separator |
| `DOT` | `.` | field access `$item.file` |
| `LPAREN` | `(` | tool call arguments |
| `RPAREN` | `)` | tool call arguments |
| `DASH` | `-` at the start of an indented list item (after leading whitespace) | branch arm, condition arm |
| `EQ_EQ` | `==` | condition comparator |
| `BANG_EQ` | `!=` | condition comparator |
| `LT` | `<` | condition comparator |
| `GT` | `>` | condition comparator |
| `LT_EQ` | `<=` | condition comparator |
| `GT_EQ` | `>=` | condition comparator |
| `NEWLINE` | `\n` (after LF normalization) — significant only at the end of a non-blank logical line | |
| `INDENT` | Synthetic — emitted by indentation algorithm (§2.3) | |
| `DEDENT` | Synthetic — emitted by indentation algorithm (§2.3) | |
| `EOF` | End of token stream | |

**Reserved keywords** (may not be used as `IDENT`):  
`pipeline`, `step`, `model`, `context`, `tool`, `prompt`, `output`, `branch`, `loop`, `condition`, `on`, `retrieve`, `summary`, `budget`, `for`, `in`, `tokens`, `default`

---

## 3. Grammar (EBNF)

The following grammar uses EBNF notation where:
- `::=` denotes production rules
- `|` denotes alternation
- `?` denotes zero or one occurrence
- `*` denotes zero or more occurrences
- `+` denotes one or more occurrences
- `( ... )` denotes grouping
- `'...'` denotes literal tokens
- `UPPER_CASE` denotes terminal tokens from §2.4

```ebnf
program          ::= pipeline_def EOF

pipeline_def     ::= 'pipeline' IDENT ':' NEWLINE INDENT pipeline_body DEDENT

pipeline_body    ::= pipeline_option* step_def+

pipeline_option  ::= model_decl
                   | budget_decl

model_decl       ::= 'model' IDENT ':' model_name NEWLINE

model_name       ::= STRING
                   | IDENT

budget_decl      ::= 'context' 'budget' ':' NUMBER 'tokens' NEWLINE

step_def         ::= 'step' IDENT ':' NEWLINE INDENT step_body DEDENT

step_body        ::= step_item+

step_item        ::= model_use
                   | context_decl
                   | prompt_decl
                   | tool_call
                   | output_decl
                   | branch_decl
                   | loop_decl
                   | condition_decl

model_use        ::= 'model' ':' IDENT NEWLINE

context_decl     ::= 'context' ':' context_sources NEWLINE

context_sources  ::= DOLLAR_IDENT
                   | '[' context_item (',' context_item)* ']'

context_item     ::= DOLLAR_IDENT
                   | retrieve_call

retrieve_call    ::= 'retrieve' '(' STRING ')'

prompt_decl      ::= 'prompt' ':' STRING NEWLINE

tool_call        ::= 'tool' ':' tool_expr NEWLINE

tool_expr        ::= IDENT '.' IDENT '(' arg_list? ')'

arg_list         ::= arg (',' arg)*

arg              ::= STRING
                   | DOLLAR_IDENT
                   | field_access
                   | NUMBER

field_access     ::= DOLLAR_IDENT '.' IDENT ( '.' IDENT )*

output_decl      ::= 'output' ':' ( DOLLAR_IDENT | summary_call ) NEWLINE

summary_call     ::= 'summary' '(' STRING ')'

branch_decl      ::= 'branch' ':' NEWLINE INDENT branch_arm+ DEDENT

branch_arm       ::= '-' 'on' ':' IDENT '->' IDENT NEWLINE

condition_decl   ::= 'condition' ':' NEWLINE INDENT condition_arm+ DEDENT

condition_arm    ::= '-' 'on' ':' condition_lhs cmp_op condition_rhs '->' IDENT NEWLINE

condition_lhs    ::= field_access
                   | DOLLAR_IDENT

condition_rhs    ::= NUMBER
                   | STRING

cmp_op           ::= '=='
                   | '!='
                   | '<'
                   | '>'
                   | '<='
                   | '>='

loop_decl        ::= 'loop' ':' 'for' DOLLAR_IDENT 'in' DOLLAR_IDENT NEWLINE
                     INDENT step_body DEDENT
```

### 3.1 Grammar Notes

**`model_name`**: When a bare `IDENT` (not a `STRING`) appears as a model name, it is treated as a symbolic alias for a model declared elsewhere in the pipeline header (see §4.5, Rule 5). String literals in `model_decl` are the actual model identifiers sent to the provider.

**`loop_decl` nesting**: `loop_decl` is a `step_item`, and `step_body` contains `step_item+`; therefore loops may nest. Maximum nesting depth is 8 levels. Exceeding this limit is a semantic error.

**`branch_arm` vs `condition_arm`**: `branch_decl` matches on a symbolic status value (e.g., `approved`, `rejected`). `condition_decl` matches on a numeric or string field using a comparison operator. The two forms may not appear in the same step.

**`field_access` in `arg_list`**: When a `field_access` appears as a tool argument, it is evaluated at runtime by traversing the type's field structure. The type checker verifies field existence at compile time (error E003).

---

## 4. Type System

### 4.1 Built-in Types

| Type | Description | JSON Representation |
|---|---|---|
| `String` | UTF-8 string, arbitrary length | `"hello"` |
| `Number` | 64-bit IEEE 754 double | `42`, `3.14` |
| `Boolean` | Logical true/false | `true`, `false` |
| `File` | Single file entry | `{"path": String, "size": Number, "mtime": String}` |
| `FileList` | Ordered list of `File` entries | `[{...}, ...]` |
| `FileContent` | Full text content of a file | `{"path": String, "content": String, "size": Number}` |
| `WriteResult` | Confirmation of a successful write | `{"path": String, "bytes_written": Number}` |
| `ExecResult` | Output of a shell command | `{"stdout": String, "stderr": String, "exit_code": Number}` |
| `ApprovalResult` | Human checkpoint decision | `{"status": "approved" \| "rejected", "notes": String}` |
| `DocumentChunk` | Retrieved document fragment | `{"content": String, "metadata": Object}` |
| `HttpResponse` | HTTP response envelope | `{"body": String, "status": Number, "headers": Object}` |
| `ModelResponse` | LLM completion output | `{"content": String, "model": String, "tokens_used": Number}` |
| `Any` | Untyped / dynamically typed | — |
| `Object` | Untyped key-value map | `{...}` |
| `List<T>` | Homogeneous ordered list of `T` | `[...]` |
| `Null` | Absent value | `null` |

`FileList` is a type alias for `List<File>`. The type checker treats `FileList` and `List<File>` as interchangeable.

### 4.2 Tool Signatures (Formal)

```
fs.glob        : (pattern: String) → FileList
fs.read        : (path: String) → FileContent
fs.write       : (path: String, content: String) → WriteResult
shell.run      : (cmd: String) → ExecResult
human.checkpoint : (data: Any) → ApprovalResult
retrieve       : (path: String) → DocumentChunk
http.get       : (url: String, headers?: Object) → HttpResponse
http.post      : (url: String, body: Any, headers?: Object) → HttpResponse
summary        : (template: String) → String
```

Parameters marked `?` are optional. Passing an unexpected additional argument is error E002. Passing a value of the wrong type for a typed parameter is error E002.

### 4.3 Type Inference Rules

The type checker assigns types to variables using the following inference rules, applied in topological order over the data dependency graph.

**Rule T-TOOL**: If a step has `tool: ns.name(args...)` and `output: $var`, then:
```
$var : return_type(ns.name)
```
where `return_type` is looked up in the tool signature table (§4.2).

**Rule T-LLM**: If a step has `prompt:` and `model:` (or defaults to `model: default`) and `output: $var`, then:
```
$var : ModelResponse
```

**Rule T-LOOP-ITEM**: If a `loop` iterates `for $item in $list` and `$list : List<T>`, then within the loop body:
```
$item : T
```
If `$list : FileList` (i.e., `List<File>`), then `$item : File`.

**Rule T-FIELD**: If `$var : T` and `T` has a field `f` of type `U`, then:
```
$var.f : U
```
Field resolution uses the JSON Representation from §4.1. Accessing `$var.f` where `f` is not a declared field of `T` is error E003.

**Rule T-SUMMARY**: If `output: summary(template)`, then:
```
$var : String
```

**Rule T-CONTEXT**: Variables referenced in `context:` must have already been assigned (Rule T-TOOL or T-LLM applied to an earlier step). Forward references in `context:` are error E001.

**Rule T-RETRIEVE**: `retrieve(path)` is syntactic sugar equivalent to `CALL(retrieve, [path])` and produces type `DocumentChunk`. It may appear inline in a `context_sources` list.

### 4.4 Type Error Code Table

| Code | Name | Description | Phase |
|---|---|---|---|
| E001 | UndefinedVariable | `$var` referenced before any assignment in the current or enclosing scope | Type Check |
| E002 | TypeMismatch | Expression of type `T2` passed where type `T1` is expected | Type Check |
| E003 | MissingField | Field `f` does not exist on type `T` | Type Check |
| E004 | UnknownTool | Tool `ns.name` is not in the stdlib and not registered as an MCP tool | Symbol Resolution |
| E005 | UnknownModel | Model alias `x` referenced in a step but not declared in the pipeline header | Symbol Resolution |
| E006 | UndefinedStep | Step name referenced in a `branch` or `condition` arm is not defined in this pipeline | Symbol Resolution |
| E007 | BackEdgeDetected | A `condition` or `branch` target step appears earlier in topological order, creating a cycle (warning, not error — infinite loop possible) | Graph Analysis |
| E008 | ShadowedVariable | A loop-scoped `$item` variable has the same name as a pipeline-level variable | Scope Check |
| E009 | UnusedVariable | `$var` is assigned by a step but never referenced in any subsequent step, context, prompt, or template | Dead Code |
| E010 | BudgetExceeded | Static estimate of context size for a step exceeds the declared `context budget` | Budget Analysis |
| E011 | MissingOutput | A step contains `prompt:` but no `output:` declaration | Completeness |
| E012 | ToolCallWithoutOutput | A step contains `tool:` but no `output:` declaration (the tool result is silently discarded) | Completeness |
| E013 | InvalidTemplate | A `{{ ... }}` template expression could not be parsed or contains an unsupported expression form | Parse |
| E014 | AmbiguousBranch | A step is reachable as the target of branch arms from more than one distinct predecessor | Graph Analysis |
| E015 | CyclicDependency | A non-loop data dependency cycle is detected (variable A depends on B depends on A) | Graph Analysis |

**Severity levels**: E007, E009, E012, E014 are warnings (compilation proceeds, IR is emitted). All others are errors (compilation halts, no IR emitted).

### 4.5 Semantic Rules

The following rules are enforced by the semantic analysis pass, after parsing and type inference are complete.

1. **Non-empty pipeline**: A pipeline must declare at least one `step`. A pipeline body containing only `model_decl` and `budget_decl` entries (no steps) is rejected.

2. **Closed branch targets**: Every step name that appears as the target of a `branch` or `condition` arm must be defined as a `step` within the same `pipeline`. Cross-pipeline references are not permitted in v0.1. Violation: E006.

3. **Loop scope isolation**: Variables declared with `output:` inside a `loop_decl` body are scoped to that loop iteration. They are not accessible outside the loop body. Accessing a loop-scoped variable outside its loop is E001.

4. **Tool/LLM exclusivity**: A step may contain either a `tool:` declaration or a `prompt:`+`model:` pair, but not both at the top level of the same step body. A step containing both `tool:` and `prompt:` at the same indentation level is a parse error. (A loop body inside a step may contain `prompt:` steps; this refers to the immediate step_items only.)

5. **Model alias scope**: A `model:` reference in a step body (`model_use`) must match an `IDENT` declared in a `model_decl` in the same pipeline header. The identifier `default` is implicitly always available as a fallback and need not be explicitly declared (though it may be overridden with `model default: ...`). Violation: E005.

6. **Default model**: If a step has `prompt:` but no `model_use`, the compiler inserts an implicit `model_use` referencing the `default` alias. If no `model default:` declaration exists in the pipeline header, a compile-time warning is emitted and the compiler falls back to `claude-3-5-sonnet`.

7. **Step ordering**: Steps are executed in declaration order unless `branch`, `condition`, or `loop` instructions redirect control flow. The compiler assigns each step a unique zero-based `step_index` in declaration order.

8. **Output uniqueness**: A variable name may appear in `output:` at most once per scope level. Reassignment of a pipeline-level output variable is disallowed (use loop-scoped variables for per-iteration outputs). Violation: E008 (shadowing) or a re-assignment parse error.

---

## 5. Stdlib Tool Reference

### 5.1 `fs.glob`

**Signature**: `fs.glob(pattern: String) → FileList`

**Description**: Recursively matches files using a glob pattern relative to the pipeline's working directory. Supports `*` (single-level wildcard), `**` (recursive wildcard), and `?` (single-character wildcard). The working directory is the repository root unless overridden in `conductor.yaml`.

**Example**:
```agent
step Ingest:
  tool: fs.glob("./src/**/*.ts")
  output: $source_files
```

**Output type details**: Returns `FileList` (a `List<File>` where each `File` has `path: String`, `size: Number` (bytes), `mtime: String` (ISO 8601 UTC)). Returns an empty list `[]` if no files match — this is not an error.

**Error conditions**: Timeout after 30 s → runtime error `TOOL_TIMEOUT`. Pattern containing null bytes → `INVALID_ARG`.

---

### 5.2 `fs.read`

**Signature**: `fs.read(path: String) → FileContent`

**Description**: Reads the complete content of a file from disk. The path may be absolute or relative to the working directory.

**Output type details**: `FileContent` has `path: String`, `content: String` (full UTF-8 text), `size: Number` (bytes). Maximum file size: 10 MiB. Files exceeding the limit produce `FILE_TOO_LARGE`.

**Error conditions**: File not found → `FILE_NOT_FOUND`. Permission denied → `PERMISSION_DENIED`. Encoding error → `ENCODING_ERROR` (file is not valid UTF-8).

---

### 5.3 `fs.write`

**Signature**: `fs.write(path: String, content: String) → WriteResult`

**Description**: Writes a string to a file atomically (write to `<path>.conductor.tmp`, then rename). Creates parent directories if they do not exist. Overwrites existing files without warning.

**Output type details**: `WriteResult` has `path: String`, `bytes_written: Number`.

**Error conditions**: Disk full → `DISK_FULL`. Permission denied → `PERMISSION_DENIED`.

---

### 5.4 `shell.run`

**Signature**: `shell.run(cmd: String) → ExecResult`

**Description**: Executes a shell command in an isolated Docker container (see Runtime Spec §7 for isolation details). The command is passed to `/bin/sh -c`. Does not inherit environment variables from the host; receives only `PATH`, `HOME`, and tenant-defined env vars from `conductor.yaml`.

**Output type details**: `ExecResult` has `stdout: String`, `stderr: String`, `exit_code: Number` (integer).

**Error conditions**: Timeout (default 300 s) → `TOOL_TIMEOUT` with `exit_code: -1`. Container start failure → `CONTAINER_ERROR`.

---

### 5.5 `human.checkpoint`

**Signature**: `human.checkpoint(data: Any) → ApprovalResult`

**Description**: Suspends pipeline execution and emits a `checkpoint` SSE event containing `data` for display in the Conductor web UI or CLI. Execution resumes when a human POSTs to `/v1/runs/{run_id}/approve`. The pipeline run transitions to `PAUSED` state.

**Output type details**: `ApprovalResult` has `status: "approved" | "rejected"`, `notes: String` (human-provided comment, may be empty string).

**Error conditions**: Run timeout before human response → `CHECKPOINT_TIMEOUT`. Run cancelled during pause → `RUN_CANCELLED`.

---

### 5.6 `retrieve`

**Signature**: `retrieve(path: String) → DocumentChunk`

**Description**: Loads a document from the pipeline's context bundle (local repo files or pre-indexed wiki). The `path` is relative to the bundle root. The result is cached per-path for 60 seconds within a run to avoid redundant I/O.

**Output type details**: `DocumentChunk` has `content: String`, `metadata: Object` (may include `title`, `source`, `last_modified` depending on the bundle source).

**Error conditions**: Path not found → `RETRIEVE_NOT_FOUND`. Bundle not configured → `BUNDLE_MISSING`.

---

### 5.7 `http.get` / `http.post`

**Signatures**:
```
http.get  : (url: String, headers?: Object) → HttpResponse
http.post : (url: String, body: Any, headers?: Object) → HttpResponse
```

**Description**: Makes outbound HTTP requests. Available only when `allow_network: true` is set in `conductor.yaml` (see Runtime Spec §7). The `body` for `http.post` is serialized as JSON.

**Output type details**: `HttpResponse` has `body: String`, `status: Number` (HTTP status code), `headers: Object` (response headers as key-value map).

**Error conditions**: DNS failure → `NETWORK_ERROR`. TLS verification failure → `TLS_ERROR`. Timeout (default 30 s) → `TOOL_TIMEOUT`. Network blocked (default) → `NETWORK_BLOCKED`.

---

### 5.8 `summary`

**Signature**: `summary(template: String) → String`

**Description**: Evaluates a template string (see §6) and returns the resulting string. Used exclusively in `output:` declarations to produce a human-readable final output for the run.

**Error conditions**: Template expression evaluation failure → `TEMPLATE_ERROR` (wraps E013).

---

## 6. Template Interpolation

Template expressions appear inside `prompt:` string values and `summary()` arguments. They use double-brace syntax: `{{ expr }}`.

### 6.1 Syntax

```ebnf
template_string  ::= ( literal_char | template_expr )*
template_expr    ::= '{{' whitespace? expr whitespace? '}}'
expr             ::= fn_call | field_access | DOLLAR_IDENT
fn_call          ::= IDENT '(' expr ')'
field_access     ::= DOLLAR_IDENT ( '.' IDENT )+
```

### 6.2 Supported Expression Forms

| Form | Example | Semantics |
|---|---|---|
| Variable reference | `{{ $dead_items }}` | Serialize the entire variable to its string representation |
| Field access | `{{ $item.line }}` | Access field `line` on `$item`; serialize to string |
| Nested field access | `{{ $item.metadata.source }}` | Traverse nested fields |
| `len()` call | `{{ len($dead_items) }}` | Return the integer length of a `List<T>` as a string |

### 6.3 Type Rules for Templates

- `{{ $var }}` where `$var : T` — valid for any `T`; the runtime uses `JSON.stringify` for complex types and `.toString()` for primitives.
- `{{ $var.f }}` where `$var : T` — valid only if `T` declares field `f` (checked at compile time; E003 if missing).
- `{{ len($var) }}` where `$var : List<T>` — valid; type error E002 if `$var` is not a list type.
- Template expressions in `prompt:` strings are evaluated at runtime immediately before the LLM call is dispatched.
- An unparseable template expression (malformed `{{ }}`) is error E013.

### 6.4 Escaping

To include a literal `{{` in a string without triggering template interpolation, use `\{{`. Similarly `\}}` for a literal `}}`.

---

## 7. Indentation Rules (Summary)

| Rule | Behavior |
|---|---|
| Unit | 2 spaces per indentation level |
| Tabs | Rejected — emit `E-INDENT` and halt lexing |
| Mixed | Each logical level must be exactly 2 spaces deeper than the enclosing level |
| Trailing whitespace | Silently discarded |
| Blank lines | Do not contribute to indentation state; generate no `NEWLINE` token |
| Maximum depth | 16 levels (32 leading spaces). Exceeding this limit is `E-INDENT`. |

---

## 8. Complete Example with Annotations

```agent
# Pipeline header: declares the pipeline name and global options
pipeline RefactorDeadCode:
  model default: claude-3-5-sonnet   # model_decl: alias "default"
  model codegen: gpt-4o              # model_decl: alias "codegen"
  context budget: 32k tokens         # budget_decl: 32768 tokens

  # Step with tool call — $source_files : FileList
  step Ingest:
    tool: fs.glob("./src/**/*.ts")
    output: $source_files

  # Step with LLM call — $dead_items : ModelResponse
  step FindDeadCode:
    model: default
    context: [$source_files, retrieve("wiki/concepts/symbol-table.md")]
    prompt: "Identify unused exports and dead branches. Output JSON: [{file, line, reason}]."
    output: $dead_items

  # Step with human checkpoint — $checkpoint_result : ApprovalResult
  step ConfirmWithUser:
    tool: human.checkpoint($dead_items)
    output: $checkpoint_result
    branch:
      - on: approved -> RefactorEach
      - on: rejected -> Abort

  # Step with loop — $item : ModelResponse (the LLM output per item)
  step RefactorEach:
    model: codegen
    loop: for $item in $dead_items
      context: [fs.read($item.file), $item.reason]
      prompt: "Remove dead code at line {{ $item.line }}. Preserve all tests."
      output: $patched_file
      tool: fs.write($item.file, $patched_file)

  # Condition on numeric field
  step Validate:
    tool: shell.run("npm test")
    output: $test_result
    condition:
      - on: $test_result.exit_code == 0 -> Done
      - on: $test_result.exit_code != 0 -> FindDeadCode   # back-edge: E007 warning

  step Done:
    output: summary("Refactor complete. {{ len($dead_items) }} items removed.")

  step Abort:
    output: summary("User rejected changes. No files written.")
```
