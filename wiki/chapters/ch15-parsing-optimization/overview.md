---
type: Chapter Section
title: "Ch 15 — Overview"
description: Mark-sweep, reference counting, generational GC.
tags: [overview, gc]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

Three families covered:

- **Mark-sweep** — walk roots, mark reachable, sweep unmarked.
- **Reference counting** — per-object counter, free on zero;
  doesn't handle cycles.
- **Generational** — split heap into young/old, collect young
  often.

J0's VM would need a root-set protocol (which stack slots are
pointers, which heap fields are pointers) before any of these would
work.
