---
type: Phase Section
title: "Phase 5 — Overview"
description: What this phase covers — running the complete J0 compiler from source file through the VM interpreter or native x86-64 binary.
tags: [phase, full-pipeline, vm, native, overview]
timestamp: 2026-06-27T00:00:00Z
---

# Overview — Phase 5: Full pipeline

## Position in the onboarding sequence

```
Phase 4: Extend J0 ──▶ [Phase 5: Full pipeline]
                                 │
                    ┌────────────┴───────────┐
                    ▼                        ▼
           VM interpreter path       x86-64 native path
           (Ch 3–9, no GCC)         (Ch 3–9 + Ch 13, needs GCC)
```

## What you do

Follow the book from Ch 4 through Ch 9 (and optionally Ch 13) to add
the parser, AST, symbol table, type checker, TAC generator, and VM
interpreter on top of the Ch 3 lexer from Phase 3. Then compile
`hello.java` end-to-end.

This phase is more expansive than the others — it covers five
additional chapters. Use the [roadmap](../../roadmap.md) and the
[wiki](../../../wiki/index.md) as companions while working through
each chapter.

## Why this matters

Phases 1–4 build a scanner. Phase 5 builds a complete compiler. After
this phase you can execute arbitrary J0 programs, understand the
full compiler architecture, and meaningfully extend any layer.

## Outputs of this phase

- `ch9/` builds and `java j0 ../ch3/hello.java` prints `hello, world`.
- (Optional) `ch13/` builds and `./a.out` prints `hello, world`.

## Time estimate

2–8 hours spread across Ch 4–9 (and optionally Ch 13), depending on
pace and reading depth.
