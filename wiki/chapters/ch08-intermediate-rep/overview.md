---
type: Chapter Section
title: "Ch 8 — Overview"
description: From typed AST to linear three-address code.
resource: /ch8/
tags: [overview, tac, ir]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

A new `tac` class represents one instruction; a new `address` class
represents one operand (region + offset). The AST grows an `icode`
list and a `gencode()` method that fills it.

## Position in the pipeline

```
typed AST (ch7) ──▶ [Ch 8 gencode] ──▶ TAC list ──▶ [Ch 9 VM] or [Ch 12 bytecode] or [Ch 13 x86-64]
```

## Outputs

- `tree.icode` — `ArrayList<tac>` of lowered instructions per
  procedure.
- Pseudo-ops `proc` / `end` / `.code` / `.global` / `.string` /
  `LAB` for region/label management.
