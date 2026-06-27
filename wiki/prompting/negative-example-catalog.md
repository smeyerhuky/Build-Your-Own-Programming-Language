---
type: Edge Case Catalog
title: Cross-chapter negative-example catalog
description: Tagged index of every edge-case entry across the per-chapter exercises/edge-cases.md files. Use to assemble themed few-shot bundles.
tags: [edge-cases, catalog, llm, debugging]
timestamp: 2026-06-27T00:00:00Z
---

# Negative-example catalog

Every per-chapter `exercises/edge-cases.md` is a source. This catalog
groups entries by **bug class** so you can pull a themed bundle.

## By bug class

### Lexer ambiguity
- `ch03-lexical-analysis` — keyword vs. identifier ordering.
- `ch03-lexical-analysis` — greedy string and unterminated comments.

### Grammar conflicts
- `ch04-syntax-analysis` — dangling-else shift/reduce.
- `ch04-syntax-analysis` — left- vs. right-recursive lists.
- `ch04-syntax-analysis` — reduce/reduce after node-type merge.

### AST malformation
- `ch05-ast` — missing `$$ = $1` passthrough.
- `ch05-ast` — broken DOT output from reused serial ids.

### Scope and lookup
- `ch06-symbol-tables` — shadowing across blocks.
- `ch06-symbol-tables` — forward reference into an unopened scope.

### Type coercion
- `ch07-type-system` — int↔double silent in expressions, illegal in
  assignment.
- `ch07-type-system` — array-dimension mismatch.

### IR lowering
- `ch08-intermediate-rep` — short-circuit `&&` lowered as straight
  arithmetic.
- `ch08-intermediate-rep` — temp aliasing.

### VM execution
- `ch09-bytecode-vm` — stack imbalance per opcode.
- `ch09-bytecode-vm` — `bp` drift across `CALL` / `RETURN`.

### Bytecode format
- `ch11-bytecode-serialization` — magic-header search vs. byte-0
  check.
- `ch11-bytecode-serialization` — operand sign-extension off-by-one.

### Codegen
- `ch12-bytecode-codegen` — duplicate strings without a string table.
- `ch12-bytecode-codegen` — label collisions across procedures.

### Backend
- `ch13-native-codegen-x86` — caller-save register live across `CALL`.
- `ch13-native-codegen-x86` — spilling a callee-save before it's
  restored.

## Suggested bundle assemblies

- **Parser onboarding bundle** — Ch 3 + Ch 4 entries; teaches the
  layering trick where the lexer must list a new token *before* the
  grammar rule references it.
- **Codegen correctness bundle** — Ch 9 + Ch 12 + Ch 13 stack-frame
  entries; teaches the off-by-eight bug class.
- **End-to-end bundle** — one entry per layer for the same surface
  feature (e.g., `for` loop); good for teaching the model to localize
  bugs to a single phase.
