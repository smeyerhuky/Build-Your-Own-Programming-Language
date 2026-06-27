---
type: Chapter Section
title: "Ch 6 — Overview"
description: Building and filling nested symbol tables off the Ch 5 AST.
resource: /ch6/
tags: [overview, symbol-table]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

Three AST traversals do the work:

1. `mkSymTables()` — walks the AST creating an empty `symtab` at each
   scope-opening node (class, method, block).
2. `populateSymTables()` — inserts every declaration into the right
   table.
3. `checkSymTables()` — walks again and verifies every used name
   resolves up the parent chain.

## Position in the pipeline

```
AST (ch5) ──▶ [Ch 6 symtab traversals] ──▶ AST + scopes ──▶ [Ch 7 types]
```

## Built-ins

The Ch 6 driver installs a synthetic `System` class so
`System.out.println("hi")` resolves without a real library.
