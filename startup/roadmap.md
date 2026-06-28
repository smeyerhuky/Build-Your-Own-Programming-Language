---
type: Chapter Section
title: Roadmap
description: Recommended learning path through all 17 chapters, with time estimates, dependency arrows, and pointers to the wiki for deeper study.
tags: [startup, roadmap, learning-path, chapters]
timestamp: 2026-06-27T00:00:00Z
---

# Roadmap

After your [First build](./first-build.md) succeeds, use this page to
plan your path through the remaining chapters.

---

## Dependency graph

```
Ch 1–2  (conceptual foundation — no code)
   │
   ▼
Ch 3  Lexer  ────────────────────────────────────┐
   │                                              │ (javalex.l is reused
   ▼                                              │  by every chapter)
Ch 4  Parser                                     │
   │                                              │
   ▼                                             ◄┘
Ch 5  AST builder
   │
   ▼
Ch 6  Symbol tables
   │
   ▼
Ch 7  Type checker
   │
   ▼
Ch 8  Three-address code (TAC)
   │
   ├──▶ Ch 9   Stack-VM interpreter  (TAC-based)
   │
   ├──▶ Ch 10  Syntax coloring in an IDE  (book-only)
   │
   ├──▶ Ch 11  Bytecode format & serialization
   │       │
   │       ▼
   │    Ch 12  Bytecode code generator
   │
   └──▶ Ch 13  x86-64 native code generator
                  (also needs a C compiler + linker)

Ch 14  Preprocessors & transpilers  (book-only)
Ch 15  Garbage collection            (book-only)
Ch 16  Final thoughts                (book-only)
Ch 17  Appendix / capstone           (book-only)
```

---

## Chapter-by-chapter summary

| # | Theme | Code? | Key artifact | Depends on |
|---|---|---|---|---|
| 1 | Why build a language? | No | — | — |
| 2 | Language design | No | — | Ch 1 |
| 3 | Lexical analysis | **Yes** | `Yylex.java` from `javalex.l` | Ch 1–2 |
| 4 | Parsing | **Yes** | parser from `j0gram.y` | Ch 3 |
| 5 | AST construction | **Yes** | `tree.java` / `tree.icn` | Ch 4 |
| 6 | Symbol tables | **Yes** | `symtab.java` | Ch 5 |
| 7 | Type checking | **Yes** | `typeinfo`, `classtype`, `methodtype` | Ch 6 |
| 8 | Intermediate code (TAC) | **Yes** | `tac.java` | Ch 7 |
| 9 | Bytecode VM interpretation | **Yes** | `j0machine.java` | Ch 8 |
| 10 | IDE syntax coloring | No | — | Ch 3 |
| 11 | Bytecode serialization | **Yes** | `.j0` file format, `Op.java` | Ch 9 |
| 12 | Bytecode code generation | **Yes** | `byc.java` | Ch 11 |
| 13 | x86-64 native codegen | **Yes** | `x64.java`, `x64loc.java` | Ch 8 |
| 14 | Preprocessors / transpilers | No | — | Ch 7 |
| 15 | Garbage collection | No | — | Ch 9 |
| 16 | Final thoughts | No | — | — |
| 17 | Appendix / capstone | No | — | — |

---

## Recommended learning paths

### Path A — Full compiler (recommended for students)

Work through **every** chapter in numeric order. Each chapter builds
directly on the previous one, and the book text explains the *why* at
each step.

```
Ch 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10 → 11 → 12 → 13
```

Then read Ch 14–17 as reflective material after you have the compiler
working end-to-end.

---

### Path B — Fast path to a working interpreter

Skip the book-only chapters and sprint to a running VM:

```
Ch 3 → 4 → 5 → 6 → 7 → 8 → 9
```

At the end of Ch 9 you have a complete front-end + stack-VM interpreter.
`hello.java` runs; you can extend J0 with new statements.

---

### Path C — Native-code focus

If you are mainly interested in how compilers target real hardware:

```
Ch 3 → 4 → 5 → 6 → 7 → 8 → 13
```

Ch 13 (`x64.java`) emits x86-64 assembly directly from the annotated
AST produced by Ch 7. You need a C compiler and linker to assemble the
output.

---

### Path D — Bytecode VM focus

If you want to understand virtual machines and bytecode formats (JVM-style):

```
Ch 3 → 4 → 5 → 6 → 7 → 8 → 9 → 11 → 12
```

Ch 11 defines the 23-opcode `.j0` bytecode format; Ch 12 generates it
from the AST.

---

## Going deeper

Each chapter has a corresponding wiki folder with four sections:

| Section | What it covers |
|---|---|
| `overview.md` | Goals, pipeline position, outputs |
| `concepts.md` | Key ideas and data structures |
| `key-files.md` | Inventory of source files and their roles |
| `deep-dive.md` | Tricky implementation details |
| `exercises/cookbook.md` | Positive examples (add a keyword, a statement, …) |
| `exercises/edge-cases.md` | Negative examples (common bugs and how to spot them) |

Cross-chapter concepts (AST design, type systems, register allocation,
…) live in [`/wiki/concepts/`](/wiki/concepts/index.md).

LLM-prompting templates and a catalog of negative examples across all
chapters are in [`/wiki/prompting/`](/wiki/prompting/index.md).
