---
type: Concept
title: Garbage collection
description: Reclaiming heap memory that's no longer reachable. Discussed in the book (Ch 15) but not implemented in this repo.
tags: [gc, garbage-collection, heap, runtime]
timestamp: 2026-06-27T00:00:00Z
---

# Garbage collection

J0's VM has a heap pointer (`hp` in `j0machine.java`) but no
collector. The book's Ch 15 walks through mark-sweep, reference
counting, and generational GC at a design level.

## What J0 has today

- Linear heap, `hp` bumped on allocation.
- No free list, no compaction, no roots tracked.

## What Ch 15 sketches

- **Mark-sweep**: tri-color marking over the root set (stack frames,
  globals, registers), then sweep the heap freeing unmarked blocks.
- **Reference counting**: per-object refcount, incremented on
  assignment, decremented on overwrite/scope-exit; cycles leak.
- **Generational**: separate young and old generations; collect young
  often, old rarely.

## Anchor

This is a book-only chapter; see
[`/wiki/chapters/ch15-parsing-optimization/index.md`](/wiki/chapters/ch15-parsing-optimization/index.md).
