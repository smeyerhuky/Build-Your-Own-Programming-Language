---
type: Chapter Section
title: "Ch 5 — Overview"
description: From parse-tree to AST; the data structure every later phase uses.
resource: /ch5/
tags: [overview, ast]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

Yacc's `$$` slot now holds a `tree` node. The grammar's semantic
actions assemble nodes bottom-up. The result is an n-ary tree.

## Position in the pipeline

```
parse tree (ch4) ──▶ [Ch 5 AST building] ──▶ AST ──▶ [Ch 6 symtab]
```

## Outputs

- A complete AST per program, printable as text or as a DOT graph.
- `serial.java` for unique node ids (DOT needs them).
