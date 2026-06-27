---
type: Concept
title: Symbol table
description: Nested HashMap of declared names per scope, with parent pointers for lexical lookup.
tags: [symbol-table, scope, semantic-analysis, lookup]
timestamp: 2026-06-27T00:00:00Z
---

# Symbol table

A **symbol table** records every declared name in a scope and lets
later phases look it up. Scopes nest (global → class → method → block),
so each table holds a `parent` pointer; lookup walks the parent chain.

## Where it appears in the book

- Introduced in [Ch 6 — Symbol Tables](/wiki/chapters/ch06-symbol-tables/index.md).
- Extended in Ch 7 to carry type information; used by every code-gen
  phase to resolve identifiers and compute addresses.

## Anchor files

- `/ch6/symtab.java` — table itself (HashMap + parent + scope name).
- `/ch6/symtab_entry.java` — one row per declared name.
- `/ch6/tree.java::mkSymTables()` / `populateSymTables()` /
  `checkSymTables()` — the three AST traversals that build, fill, and
  validate the tables.

## API

| Method | Purpose |
|---|---|
| `symtab(scope)` | Construct a root table. |
| `symtab(scope, parent)` | Construct a nested table. |
| `insert(name, isConst)` | Add a name; calls `j0.semerror` on redeclaration. |
| `insert(name, isConst, sub)` | Add a name that opens a sub-scope (class, method). |
| `lookup(name)` | Local-only lookup; returns `null` on miss. Walks parents by re-calling on `parent` in `tree.java`. |
| `print(level)` | Pretty-print, indented. |

## Common bugs

- Calling `insert` twice for the same name crashes with `redeclaration
  of X`; this is by design.
- Doing a single-table lookup where a chain walk is needed produces
  spurious "undefined" errors — the canonical fix is `for (s = this;
  s != null; s = s.parent) { if ((e = s.lookup(name)) != null) return
  e; }`.

See [`/wiki/chapters/ch06-symbol-tables/exercises/edge-cases.md`](/wiki/chapters/ch06-symbol-tables/exercises/edge-cases.md).
