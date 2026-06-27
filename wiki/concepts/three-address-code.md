---
type: Concept
title: Three-address code
description: IR instructions of the form `op dst, src1, src2`; the workhorse of the course's middle-end.
tags: [tac, ir, three-address-code]
timestamp: 2026-06-27T00:00:00Z
---

# Three-address code (TAC)

A TAC instruction is `op dst, src1, src2`. Complex expressions are
**lowered** into a sequence of TAC by introducing temporaries:

```
t1 = b * c
t2 = a + t1
d  = t2
```

## Pseudo-ops vs. real ops

Some `tac` records aren't arithmetic; they're directives:

- `proc <region>,<param-bytes>,<local-bytes>` — open a procedure.
- `end` — close a procedure.
- `.code`, `.global`, `.string` — region markers, like assembler
  pseudo-ops.
- `LAB L<n>` — label declaration (`print()` renders it as `Ln:`).

This vocabulary is enforced by the `switch (op)` in `/ch9/tac.java`.

## Where it appears in the book

- Introduced in [Ch 8 — Intermediate Code Generation](/wiki/chapters/ch08-intermediate-rep/index.md).
- Lowered to bytecode in Ch 9 / Ch 12 and to x86-64 in Ch 13.

## Common bugs

- Forgetting to allocate a fresh temp address for each subexpression
  produces *aliased* registers and clobbers values.
- Short-circuit evaluation of `&&` / `||` requires a control-flow
  lowering with `BIF` / `GOTO`, not a straight-line `AND`/`OR`.
