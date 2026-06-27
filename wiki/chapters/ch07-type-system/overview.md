---
type: Chapter Section
title: "Ch 7 — Overview"
description: Type representation and the three AST passes that use it.
resource: /ch7/
tags: [overview, types]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

J0 distinguishes primitives, arrays, classes, and method types.
Every AST node grows a `typeinfo` field; three passes do the work:

- `calctype()` — bottom-up: compute the natural type of each
  expression.
- `checktype()` — verify operands match the operator/context.
- `assigntype()` — propagate types into context-dependent nodes
  (e.g., the `null` literal needs a target type to mean anything).

## Position in the pipeline

```
AST + scopes (ch6) ──▶ [Ch 7 type passes] ──▶ typed AST ──▶ [Ch 8 TAC]
```

## Outputs

- `tree.typ` set on every AST node.
- `j0.semerror` emitted on type mismatches.
