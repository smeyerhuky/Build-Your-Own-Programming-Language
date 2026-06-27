---
type: Chapter Section
title: "Ch 5 — Deep dive: tree.java"
description: The n-ary AST node and its three printers.
resource: /ch5/tree.java
tags: [deep-dive, ast, tree, dot]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `tree.java`

The Java AST node is a thin n-ary structure:

```java
class tree {
   String sym;          // category, e.g. "AddExpr+"
   token tok;           // non-null iff this is a leaf
   ArrayList<tree> kids;
   int id;              // from serial.next(), used by print_graph
   // populated later: symtab st, typeinfo typ, address addr
}
```

## Constructors

A handful of overloads: empty, single-token leaf, and "category + N
children" — Yacc actions call the latter:

```
AddExpr : AddExpr '+' MulExpr   { $$ = new tree("AddExpr+", $1, $3); }
        | MulExpr               { $$ = $1; }
        ;
```

Note the **passthrough**: the single-RHS rule uses `$$ = $1` so the
intermediate `AddExpr` doesn't add a useless wrapper node.

## Printing

- `print()` — indented textual dump for humans.
- `print_graph()` — emits DOT (graphviz) statements: one `node[N]`
  line per AST node, one edge per child. The `serial.next()` id is
  what makes each `node[N]` unique.

## Cross-references

- [Concept: AST](/wiki/concepts/abstract-syntax-tree.md)
- [Tool: Yacc / Iyacc](/wiki/tools/yacc-iyacc.md)
