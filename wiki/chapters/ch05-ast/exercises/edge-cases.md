---
type: Edge Case Catalog
title: "Ch 5 — Edge cases"
description: Common AST-construction bugs.
resource: /ch5/tree.java
tags: [edge-cases, cot, ast]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 5 AST

## 1. Missing passthrough

### Buggy input
```
AddExpr : AddExpr '+' MulExpr  { $$ = new tree("AddExpr+", $1, $3); }
        | MulExpr              { /* forgot $$ = $1 */ }
        ;
```

### Symptom
Every AST that doesn't use `+` has a `null` AddExpr child somewhere.
Later phases see `NullPointerException` deep in `tree.print()` or
`calctype()`.

### CoT trace
1. Yacc's `$$` defaults to `$1` *only* if the action block is
   absent. An empty action block leaves `$$` uninitialized.
2. The `MulExpr` value never reaches the parent rule.
3. Fix by adding `$$ = $1;` explicitly.

### Fix
```
        | MulExpr              { $$ = $1; }
```

### Lesson
If you write an action block, also write `$$ = $1` in passthrough
alternations. Don't rely on Yacc's default — it's only the default
when there's *no* action at all.

---

## 2. Reused `serial` id

### Buggy input
Someone reuses a captured `id` across two `new tree(...)` calls
because they wrote `tree t1, t2; t1.id = t2.id = serial.next();`.

### Symptom
GraphViz renders only one node where two were expected, because
two DOT statements have the same `node[N]` identifier.

### CoT trace
1. Every AST node needs a *unique* id.
2. Sharing ids collapses two nodes into one in the rendered graph.
3. Fix by calling `serial.next()` once per node.

### Fix
Always call `serial.next()` in the `tree` constructor; never
re-assign `id` after construction.

### Lesson
Identity is a constructor concern. Mutate-after-construct on `id` is
a recipe for graph collisions.

---

## 3. Concrete-syntax leak into the AST

### Buggy input
```
IfThenStmt : IF '(' Expr ')' Block
             { $$ = new tree("If", $1, $2, $3, $4, $5); }
           ;
```

### Symptom
The AST has nodes labeled `(`, `)`, and `if` cluttering the output.
Later traversal code has to special-case them.

### CoT trace
1. `$1` is the `IF` token, `$2` is `(`, `$4` is `)` — all noise.
2. Only `$3` (Expr) and `$5` (Block) belong in the AST.
3. Drop the noise children.

### Fix
```
IfThenStmt : IF '(' Expr ')' Block
             { $$ = new tree("If", $3, $5); }
           ;
```

### Lesson
AST = abstract = only semantic children. Use Yacc's `$N` to **pick**,
not to mindlessly forward all values.

---

## 4. Sharing a child between two parents

### Buggy input
```
SomeStmt : Expr Expr   { $$ = new tree("Twin", $1, $1); }
         ;
```

### Symptom
A later mutating traversal (`assigntype`) walks the shared subtree
twice, producing wrong-looking types.

### CoT trace
1. Sharing a node makes the structure a DAG, not a tree.
2. Mutating traversals assume tree-shape and visit each node once.
3. Either clone the subtree or restructure to avoid sharing.

### Fix
```
SomeStmt : Expr Expr   { $$ = new tree("Twin", $1, $2); }
         ;
```

### Lesson
"AST" means tree. The compiler is free to lower to a DAG later
(common-subexpression elimination), but the initial AST should be a
strict tree.
