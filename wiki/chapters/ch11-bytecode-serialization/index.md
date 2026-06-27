---
type: Chapter
title: "Ch 11 — Bytecode Interpreters / Serialization"
description: Serialize bytecode to a .j0 file, locate the "Jzero!!\0" magic, and run it from disk.
resource: /ch11/
tags: [chapter, bytecode-format, magic-header, j0machine, j0x]
timestamp: 2026-06-27T00:00:00Z
---

# Ch 11 — Bytecode Interpreters / Serialization

The bytecode is now a *file*. A separate `j0x` program loads,
verifies, and runs it.

## Children

- [Overview](/wiki/chapters/ch11-bytecode-serialization/overview.md)
- [Concepts](/wiki/chapters/ch11-bytecode-serialization/concepts.md)
- [Key files](/wiki/chapters/ch11-bytecode-serialization/key-files.md)
- [Deep dive: file format & `j0machine`](/wiki/chapters/ch11-bytecode-serialization/deep-dive.md)
- [Exercises](/wiki/chapters/ch11-bytecode-serialization/exercises/index.md)
