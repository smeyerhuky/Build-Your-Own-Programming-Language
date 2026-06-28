---
type: Phase
title: "Phase 5 — Full pipeline"
description: Compile a J0 source file all the way through the pipeline — lexer → parser → AST → TAC → VM interpreter, and optionally through the x86-64 native backend.
tags: [phase, full-pipeline, pipeline, startup]
timestamp: 2026-06-27T00:00:00Z
---

# Phase 5 — Full pipeline

**Goal:** A J0 source file runs end-to-end through the compiler. The
VM interpreter prints the correct output. Optionally, the native
x86-64 binary also runs.

## Children

- [Overview](./overview.md)
- [Checklist](./checklist.md)
- [Concepts](./concepts.md)
- [Deep dive](./deep-dive.md)
- [Exercises](./exercises/index.md)
  - [Cookbook (positive)](./exercises/cookbook.md)
  - [Edge cases (negative)](./exercises/edge-cases.md)

## Prerequisites

Phase 3 complete. Phases 1–4 give you a stronger foundation but
Phase 5 can be attempted directly after Phase 3 by following Ch 4–9
of the book in parallel.

## Exit criteria

### Interpreter path (Ch 9)

```bash
cd ch9
make
java j0 ../ch3/hello.java
# → hello, world
```

### Native path (Ch 13, requires GCC)

```bash
cd ch13
make
./j0 ../ch3/hello.java && ./a.out
# → hello, world
```

## Pipeline stages required

| Stage | Chapter | Key file |
|---|---|---|
| Lex | Ch 3 | `javalex.l` → `Yylex.java` |
| Parse | Ch 4 | `j0gram.y` → parser |
| AST | Ch 5 | `tree.java` |
| Symbol table | Ch 6 | `symtab.java` |
| Type check | Ch 7 | `typeinfo.java` |
| TAC gen | Ch 8 | `tac.java` |
| VM interpret | Ch 9 | `j0machine.java` |
| x86-64 (opt.) | Ch 13 | `x64.java` |

## See also

- [Compiler pipeline concept](../../concepts/compiler-pipeline.md)
- [Roadmap](../../roadmap.md)
- [Wiki pipeline overview](/wiki/index.md)
