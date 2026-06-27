---
type: Edge Case Catalog
title: "Ch 4 — Edge cases (negative examples)"
description: Common grammar bugs with chain-of-thought walks.
resource: /ch4/j0gram.y
tags: [edge-cases, cot, parser, grammar, conflicts]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 4 parser

## 1. Dangling else

### Buggy input
A naive grammar:

```
IfStmt : IF '(' Expr ')' Stmt
       | IF '(' Expr ')' Stmt ELSE Stmt ;
```

### Symptom
Yacc reports `1 shift/reduce conflict` on the `ELSE` token. The
parser still works, but the resolution is "shift ELSE," which means
`else` binds to the *innermost* `if`.

### CoT trace
1. After matching `IF '(' Expr ')' Stmt`, the parser sees `ELSE` and
   has two options: shift `ELSE` (continue the second rule) or
   reduce by the first rule.
2. Yacc's default — shift — is what every C-like language wants.
3. J0 sidesteps the conflict entirely by giving `if-else` its own
   rule (`IfThenElseStmt`) and `if-else-if` its own
   (`IfThenElseIfStmt`), with `Block` (not bare `Stmt`) as the body.

### Fix
Either accept the default shift (with a 1-line comment in the
grammar) or restructure as J0 does:

```
IfThenStmt        : IF '(' Expr ')' Block ;
IfThenElseStmt    : IF '(' Expr ')' Block ELSE Block ;
```

### Lesson
Restructuring the grammar can eliminate a conflict — and a `Block`
body (forced braces) is itself a small language-design decision.

---

## 2. Reduce/reduce after merging node types

### Buggy input
Someone collapses `MethodCall` and `InstantiationExpr`:

```
PrimaryExpr : Name '(' ArgListOpt ')'  /* used to be MethodCall */
            | Name '(' ArgListOpt ')'  /* used to be InstantiationExpr */
            ;
```

### Symptom
Yacc reports `1 reduce/reduce conflict on tokens '(' ')'`.

### CoT trace
1. After matching `Name '(' ArgListOpt ')'`, two reductions apply.
2. Yacc picks the first by default, but the resolution is silent and
   wrong — every method call now reduces to "InstantiationExpr."
3. The fix is to **keep them separate at the grammar level** and let
   the semantic phase decide which one a given `Name` actually is
   (Ch 6/7 resolves `Name` to a class symbol or method symbol).

### Fix
Keep `MethodCall` and `InstantiationExpr` as separate productions, as
in the canonical `j0gram.y`.

### Lesson
Don't merge productions that look identical syntactically but mean
different things semantically. Disambiguate later, not in the
grammar.

---

## 3. Left- vs right-recursion on a list

### Buggy input
```
ArgList : Expr ',' ArgList
        | Expr ;
```

### Symptom
Yacc still accepts the grammar, but for large argument lists the
parser stack grows linearly because each `,` shift defers a reduce
until the end.

### CoT trace
1. LALR(1) parsers prefer **left**-recursive rules: each new RHS
   reduces immediately, keeping the stack small.
2. The list rule above is right-recursive — it must read the entire
   list before reducing.
3. Flipping to left-recursion fixes the stack growth.

### Fix
```
ArgList : ArgList ',' Expr
        | Expr ;
```

### Lesson
Default to left-recursion for list rules in Yacc/Iyacc. Use right-
recursion only when you specifically want right-associative
semantics.

---

## 4. Missing `%token` for a new keyword

### Buggy input
You add `"do" { return j0.scan(parser.DO); }` to the lexer and a
grammar rule `DoWhileStmt : DO Block WHILE '(' Expr ')' ';' ;`, but
forget the `%token DO` line.

### Symptom
Yacc compiles but warns `nonterminal "DO" useless in grammar`, and at
runtime the parser hits `syntax error` on any `do`.

### CoT trace
1. Without `%token`, Yacc treats `DO` as a **non-terminal** name.
2. There is no rule producing `DO`, so it can never appear.
3. The lexer returns `parser.DO`, which is now an integer Yacc never
   declared, mapped to "unknown" → syntax error.

### Fix
Add `%token DO` to the declarations section.

### Lesson
Every terminal — keyword or operator — must be declared with
`%token`. The lexer's `parser.X` constants come from Yacc's token
table; if Yacc doesn't declare it, the constant is meaningless.
