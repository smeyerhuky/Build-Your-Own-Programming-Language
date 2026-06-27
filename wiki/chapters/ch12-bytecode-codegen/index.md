---
type: Chapter
title: "Ch 12 — Generating Bytecode"
description: AST → bytecode in one pass, with a string table and label fixups.
resource: /ch12/
tags: [chapter, bytecode-codegen, byc, j0]
timestamp: 2026-06-27T00:00:00Z
---

# Ch 12 — Generating Bytecode

The compiler now emits a complete `.j0` file. The path is AST →
bytecode directly (the TAC IR is still computed internally but the
output is bytes).

## Children

- [Overview](/wiki/chapters/ch12-bytecode-codegen/overview.md)
- [Concepts](/wiki/chapters/ch12-bytecode-codegen/concepts.md)
- [Key files](/wiki/chapters/ch12-bytecode-codegen/key-files.md)
- [Deep dive: `byc.java`](/wiki/chapters/ch12-bytecode-codegen/deep-dive.md)
- [Exercises](/wiki/chapters/ch12-bytecode-codegen/exercises/index.md)
