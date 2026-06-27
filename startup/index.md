---
type: Index
title: "Getting Started — Build Your Own Programming Language"
description: Practical onboarding guide. Start here to install prerequisites, set up your environment, build Ch 3, and choose your learning path.
tags: [startup, onboarding, getting-started]
timestamp: 2026-06-27T00:00:00Z
---

# Getting Started

This folder is your fastest path from a blank machine to a running J0
compiler. Read the four pages in order, then follow the roadmap to
explore the rest of the course.

## Pages in this folder

| Page | What it covers |
|---|---|
| [Prerequisites](./prerequisites.md) | All software you must install before writing any code. |
| [Environment setup](./environment-setup.md) | Clone the repo and verify every tool is wired up correctly. |
| [First build](./first-build.md) | Compile and run the Ch 3 lexer against `hello.java`. |
| [Roadmap](./roadmap.md) | Recommended learning path through all 17 chapters. |

## What you will build

The course walks you through a complete compiler for **J0** — a small
Java subset. By the end you will have:

- a **Flex / Uflex lexer** that turns source text into tokens (Ch 3),
- a **Yacc / Iyacc parser** that turns tokens into a parse tree (Ch 4),
- an **AST** with symbol tables and type annotations (Ch 5–7),
- a **three-address-code** middle-end and a stack-VM interpreter (Ch 8–9),
- a **bytecode serializer/loader** (Ch 11–12),
- a **native x86-64 code generator** (Ch 13).

The full pipeline at a glance:

```
source.j0
   │  Ch 3  Lexer        → tokens
   │  Ch 4  Parser       → parse tree
   │  Ch 5  AST builder  → AST
   │  Ch 6  Symbol table → annotated AST
   │  Ch 7  Type checker → fully annotated AST
   │  Ch 8  TAC gen      → three-address code
   ├──▶ Ch 9  Stack VM   (executes TAC directly)
   ├──▶ Ch 11/12  Bytecode (.j0 file + interpreter)
   └──▶ Ch 13  x86-64    (native binary)
```

## Relationship to the wiki

The [`/wiki/`](/wiki/index.md) folder is a deep-reference OKF bundle —
concepts, per-chapter notes, LLM-prompting templates. Once you have
your environment working, the wiki is the place to look up *why*
something works the way it does.

This `startup/` folder is deliberately narrower: it only tells you
*what to run*.
