---
type: Cookbook
title: "Ch 4 — Cookbook (positive examples)"
description: Canonical grammar-extension patterns.
resource: /ch4/j0gram.y
tags: [cookbook, few-shot, parser, grammar]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 4 parser

## 1. Add a `do { } while` statement

### Prompt
"Add a `do { } while (e);` loop to J0."

### Reference solution
1. Lexer: `"do" { return j0.scan(parser.DO); }` (see Ch 3
   cookbook).
2. Grammar in `/ch4/j0gram.y`:
   ```
   %token DO
   ...
   Stmt : ... | DoWhileStmt ;
   DoWhileStmt : DO Block WHILE '(' Expr ')' ';' ;
   ```

### Why this is canonical
- Statement-level extension: add to `Stmt`'s alternation, then write
  one rule for the new statement.
- Use existing `Block` and `Expr` non-terminals — never recreate
  expression grammar.

---

## 2. Add a unary plus operator

### Prompt
"Allow `+x` as well as `-x` and `!x`."

### Reference solution
In `/ch4/j0gram.y`:

```
UnaryExpr : '-' UnaryExpr
          | '+' UnaryExpr
          | '!' UnaryExpr
          | PostFixExpr ;
```

### Why this is canonical
Unary operators belong on `UnaryExpr`; this is the precedence layer
they bind at.

---

## 3. Add the ternary `?:` operator

### Prompt
"Add `cond ? a : b` to J0."

### Reference solution
Insert a level between `Expr` and `CondOrExpr`:

```
%token QUESTION
...
Expr        : Conditional | Assignment ;
Conditional : CondOrExpr
            | CondOrExpr QUESTION Expr ':' Conditional ;
```

The right-recursion on the third operand gives right-associativity
(`a?b:c?d:e` ≡ `a?b:(c?d:e)`).

### Why this is canonical
- New precedence levels are *inserted* between existing ones, not
  bolted onto an existing level.
- Right-associative operators use right-recursion.

---

## 4. Allow a top-level `import` directive

### Prompt
"Let J0 programs start with `import` lines before the `class`."

### Reference solution
1. Lexer: `"import" { return j0.scan(parser.IMPORT); }`
2. Grammar:
   ```
   %token IMPORT
   Program : ImportList ClassDecl ;
   ImportList : | ImportList ImportDecl ;
   ImportDecl : IMPORT Name ';' ;
   ```
3. Replace `ClassDecl` as the start symbol with `Program` via
   `%start Program`.

### Why this is canonical
Introducing a new top-level non-terminal is the way to add anything
that lives outside `ClassDecl` without breaking existing rules.
