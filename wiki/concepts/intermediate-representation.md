---
type: Concept
title: Intermediate representation
description: A target-neutral middle form sitting between the typed AST and the final code.
tags: [ir, intermediate-representation, tac]
timestamp: 2026-06-27T00:00:00Z
---

# Intermediate representation (IR)

An **IR** decouples the front-end (lexer, parser, type checker) from
the back-end (bytecode, native). In this course the IR is **three-
address code** — see [`/wiki/concepts/three-address-code.md`](/wiki/concepts/three-address-code.md).

A good IR is:

- **Target-neutral** — no x86 registers, no JVM opcodes leak in.
- **Address-based** — every operand is an `address` descriptor with a
  *region* (`local`, `global`, `string`, `imm`, `lab`) and an offset.
- **Linear** — a list of three-address instructions, easy to walk for
  bytecode emission or instruction selection.

## Where it appears in the book

- Designed in [Ch 8 — Intermediate Code Generation](/wiki/chapters/ch08-intermediate-rep/index.md).
- Consumed by Ch 9 (VM), Ch 12 (bytecode emission), and Ch 13
  (x86-64).

## Anchor files

- `/ch9/tac.java` — instruction record (`op`, `op1`, `op2`, `op3`).
- `/ch9/address.java` — address descriptor.
