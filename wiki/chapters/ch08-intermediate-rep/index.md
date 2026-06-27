---
type: Chapter
title: "Ch 8 — Intermediate Code Generation"
description: Lower the typed AST into a list of three-address-code (TAC) instructions.
resource: /ch8/
tags: [chapter, ir, tac, three-address-code]
timestamp: 2026-06-27T00:00:00Z
---

# Ch 8 — Intermediate Code Generation

The first code-generation pass. The AST is *lowered* into a flat
list of `tac` records, each `op dst, src1, src2`.

## Children

- [Overview](/wiki/chapters/ch08-intermediate-rep/overview.md)
- [Concepts](/wiki/chapters/ch08-intermediate-rep/concepts.md)
- [Key files](/wiki/chapters/ch08-intermediate-rep/key-files.md)
- [Deep dive: `tac.java`](/wiki/chapters/ch08-intermediate-rep/deep-dive.md)
- [Exercises](/wiki/chapters/ch08-intermediate-rep/exercises/index.md)
