---
type: Cookbook
title: "Ch 5 — Cookbook"
description: Canonical AST-construction patterns.
resource: /ch5/tree.java
tags: [cookbook, ast, semantic-actions]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 5 AST

## 1. Build a node for a new binary operator

### Prompt
"Add an AST node for the bitwise-OR operator `|`."

### Reference solution
```
OrExpr : OrExpr '|' XorExpr   { $$ = new tree("OrExpr|", $1, $3); }
       | XorExpr               { $$ = $1; }
       ;
```

### Why this is canonical
- New operators get a new category name like `OrExpr|`.
- The passthrough alternation `$$ = $1` keeps the AST flat.

---

## 2. Build a node for a new statement

### Prompt
"Add an AST node for `do { } while (e);`."

### Reference solution
```
DoWhileStmt : DO Block WHILE '(' Expr ')' ';'
              { $$ = new tree("DoWhile", $2, $5); }
            ;
```

Only the `Block` and the `Expr` go into the tree — `DO`, `WHILE`,
`(`, `)`, `;` are concrete-syntax noise.

### Why this is canonical
AST nodes carry **semantic** children, not concrete tokens.

---

## 3. Add a unique-id printer

### Prompt
"Print the AST as DOT to stdout."

### Reference solution
After `parse()` returns the root:

```java
System.out.println("digraph G {");
root.print_graph();
System.out.println("}");
```

### Why this is canonical
`print_graph` is the existing primitive; wrapping it in `digraph
G { ... }` produces a complete DOT file.

---

## 4. Re-using an existing non-terminal

### Prompt
"Add a `repeat N { Block }` loop with `N` an integer literal."

### Reference solution
```
RepeatStmt : REPEAT INTLIT Block    { $$ = new tree("Repeat", $2, $3); }
           ;
```

### Why this is canonical
Reuse `INTLIT` and `Block` rather than defining new grammar — this
keeps the AST regular and lets later phases treat the literal as any
other `INTLIT`.
