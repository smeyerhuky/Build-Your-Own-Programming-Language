---
type: Phase Section
title: "Phase 5 — Concepts"
description: The cross-phase ideas the full-pipeline phase draws on.
tags: [phase, full-pipeline, concepts, pipeline, vm]
timestamp: 2026-06-27T00:00:00Z
---

# Concepts — Phase 5: Full pipeline

## The two back-ends

The J0 compiler has two independent output paths after Ch 8:

| Path | Chapters | Output | Runs via |
|---|---|---|---|
| TAC generator | Ch 9 | Prints TAC instructions | `java ch9.j0` |
| Bytecode | Ch 11–12 | `.j0` file (binary, magic `Jzero!!\0`) | `java j0x` |
| x86-64 native | Ch 13 | AT&T-syntax assembly → linked binary | `./a.out` |

Phase 5 targets the VM interpreter path (Ch 9) as the primary goal.
The native path is optional.

## TAC — Three-address code

The Ch 8 front-end emits **three-address code**: a list of
pseudo-instructions of the form `t1 = t2 op t3`. TAC is simpler than
a real machine ISA (infinite registers, no calling conventions) and
simpler than the AST (flat, no nesting). Ch 9 executes TAC directly.

See [Wiki: Three-address code](/wiki/concepts/three-address-code.md).

## Stack frame layout

The VM uses a call stack with one frame per method invocation. Each
frame holds:
- Local variables (indexed by slot number).
- A return address.
- The caller's frame pointer.

See [Wiki: Stack frame](/wiki/concepts/stack-frame.md).

## Full pipeline concept

See [Compiler pipeline](../../concepts/compiler-pipeline.md) for the
startup-flavored end-to-end view, and
[Wiki: Compiler pipeline](/wiki/concepts/compiler-pipeline.md) for
the deeper reference.
