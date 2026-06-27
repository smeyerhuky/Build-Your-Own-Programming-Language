---
type: Chapter Section
title: "Ch 9 — Overview"
description: A stack VM that executes J0 bytecode.
resource: /ch9/
tags: [overview, vm, interpreter]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

A new `j0machine` class is the VM. It owns:

- `code` / `codebuf` — byte array + `ByteBuffer` view.
- `stack` / `stackbuf` — 800 KB scratch.
- `ip`, `sp`, `bp`, `hp` — pointers.
- `op`, `opr`, `opnd` — last-fetched instruction parts.

The loop is `fetch(); switch (op) { ... }`.

## Position in the pipeline

```
TAC (ch8) ──▶ [Ch 9 bytecode emission + j0machine] ──▶ stdout
```

## Outputs

- A runnable interpreter for the J0 bytecode ISA.
- A bytecode emitter walk on the AST (in `tree.java`).
