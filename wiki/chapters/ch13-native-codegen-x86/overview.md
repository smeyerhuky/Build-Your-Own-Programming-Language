---
type: Chapter Section
title: "Ch 13 — Overview"
description: x86-64 emission with a tiny register allocator and SysV AMD64 ABI.
resource: /ch13/
tags: [overview, native, x86-64]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

A new emitter writes AT&T-syntax assembly. Three new classes:

- `x64` — one instruction record.
- `x64loc` — one operand (register, immediate, displacement, or
  label).
- `RegUse` — bitset tracking which GP registers are live.

The host toolchain (`gcc`, `as`, `ld`) assembles and links the
output.

## Position in the pipeline

```
typed AST ──▶ [Ch 13 x86-64 emission] ──▶ .s file ──▶ gcc → ELF binary
```
