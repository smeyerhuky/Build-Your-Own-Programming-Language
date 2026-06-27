---
type: Concept
title: Abstract syntax tree
description: A compact, semantically meaningful tree built from the parse tree, used by every later compiler phase.
tags: [ast, tree, semantic-actions, dot, graphviz]
timestamp: 2026-06-27T00:00:00Z
---

# Abstract syntax tree (AST)

An **AST** is a tree of nodes that captures the *meaningful* structure
of the source program, with concrete-syntax noise (parentheses,
semicolons, intermediate non-terminals that exist only for parsing)
removed. AST nodes carry a category, child list, optional token, and
fields added by later phases (symbol-table pointer in Ch 6, type in
Ch 7, address in Ch 8/9).

## Where it appears in the book

- Constructed first in [Ch 5 — Syntax Trees](/wiki/chapters/ch05-ast/index.md).
- Walked or annotated by every later chapter.

## Anchor files

- `/ch5/tree.java` (Java) — the canonical n-ary node.
- `/ch5/tree.icn` (Unicon) — same shape, smaller.
- `/ch5/serial.java` — unique-id generator used for DOT output.

## Construction

Yacc semantic actions in `j0gram.y` build nodes from RHS values:

```
AddExpr : AddExpr '+' MulExpr  { $$ = new tree("AddExpr+", $1, $3); }
        | MulExpr              { $$ = $1; }
        ;
```

The single-production case `AddExpr : MulExpr { $$ = $1; }` is what
keeps the AST smaller than the parse tree.

## Visualization

`tree.print_graph()` emits GraphViz DOT so you can render an AST as a
PNG. The serial-id mechanism ensures every node has a stable
identifier, which DOT requires.

## Common bugs

- Forgetting `$$ = $1` in a passthrough production silently makes the
  parent's child list partial and the AST malformed.
- Reusing a node across two parents creates a DAG, not a tree, and
  breaks tree traversals that mutate per node.

See [`/wiki/chapters/ch05-ast/exercises/edge-cases.md`](/wiki/chapters/ch05-ast/exercises/edge-cases.md).
