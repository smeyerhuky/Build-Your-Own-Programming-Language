---
type: Setup Concept
title: Compiler pipeline (startup view)
description: A startup-oriented end-to-end view of the J0 compiler pipeline — what each chapter adds and what the overall data flow looks like.
tags: [concept, pipeline, startup, compiler]
timestamp: 2026-06-27T00:00:00Z
---

# Compiler pipeline (startup view)

This is the practical, startup-oriented view of the pipeline. For the
full academic treatment see
[Wiki: Compiler pipeline](/wiki/concepts/compiler-pipeline.md).

## End-to-end data flow

```
hello.java  (J0 source)
   │
   ▼  Ch 3  jflex javalex.l → java simple hello.java
tokens
   │
   ▼  Ch 4  byacc j0gram.y → java j0 hello.java
parse tree
   │
   ▼  Ch 5  tree.java
AST
   │
   ▼  Ch 6  symtab.java
AST + symbol tables
   │
   ▼  Ch 7  typeinfo.java
fully annotated AST
   │
   ▼  Ch 8  tac.java
three-address code (TAC)
   │
   ├──▶ Ch 9   j0machine.java  → executes TAC → "hello, world"
   ├──▶ Ch 11  bytecode file (.j0) → j0machine → "hello, world"
   ├──▶ Ch 12  byc.java generates .j0 bytecode
   └──▶ Ch 13  x64.java → hello.s → gcc → a.out → "hello, world"
```

## What each chapter adds

| Chapter | New phase | New artifact |
|---|---|---|
| Ch 3 | Lexer | `Yylex.java` (token stream) |
| Ch 4 | Parser | `parser.java` (parse tree) |
| Ch 5 | AST builder | `tree.java` (AST) |
| Ch 6 | Symbol table | `symtab.java` (annotated AST) |
| Ch 7 | Type checker | `typeinfo.java` (typed AST) |
| Ch 8 | TAC generator | `tac.java` (TAC list) |
| Ch 9 | VM interpreter | `j0machine.java` (executes TAC) |
| Ch 11–12 | Bytecode backend | `.j0` binary + loader |
| Ch 13 | x86-64 backend | `.s` assembly + native binary |

## How to navigate the pipeline

- Building Ch N requires that Ch 3–(N-1) have been built first.
- Each chapter adds one `.java` class; the others carry forward.
- `hello.java` is the test input for every chapter.

## See also

- [Roadmap](../roadmap.md)
- [Build artifacts](./build-artifacts.md)
