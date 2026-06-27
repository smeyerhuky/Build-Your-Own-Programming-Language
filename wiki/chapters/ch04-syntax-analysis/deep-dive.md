---
type: Chapter Section
title: "Ch 4 — Deep dive: j0gram.y"
description: Walks the J0 grammar's token table, top-level rules, and the precedence layering trick.
resource: /ch4/j0gram.y
tags: [deep-dive, yacc, grammar, precedence]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `j0gram.y`

`/ch4/j0gram.y` is one of the simplest *real* CFGs in the repo —
about 100 lines covering an entire Java subset.

## Token declarations

```
%token BREAK DOUBLE ELSE FOR IF INT RETURN VOID WHILE
%token IDENTIFIER CLASSNAME CLASS STRING BOOL
%token INTLIT DOUBLELIT STRINGLIT BOOLLIT NULLVAL
%token LESSTHANOREQUAL GREATERTHANOREQUAL
%token ISEQUALTO NOTEQUALTO LOGICALAND LOGICALOR
%token INCREMENT DECREMENT PUBLIC STATIC
```

Every multi-char terminal needs a name here — Yacc treats single-char
operators (`'+'`, `'<'`) as themselves.

## Top-level rule

```
ClassDecl: PUBLIC CLASS IDENTIFIER ClassBody;
```

A J0 program is one class.

## Precedence by layering

J0 does **not** use `%left`/`%right` directives. Instead, precedence is
encoded as rule-level layering. From loose to tight:

```
Expr        : CondOrExpr | Assignment ;
CondOrExpr  : CondAndExpr | CondOrExpr LOGICALOR CondAndExpr ;
CondAndExpr : EqExpr      | CondAndExpr LOGICALAND EqExpr ;
EqExpr      : RelExpr     | EqExpr ISEQUALTO RelExpr | EqExpr NOTEQUALTO RelExpr ;
RelExpr     : AddExpr     | RelExpr RelOp AddExpr ;
AddExpr     : MulExpr     | AddExpr '+' MulExpr | AddExpr '-' MulExpr ;
MulExpr     : UnaryExpr   | MulExpr '*' UnaryExpr | MulExpr '/' UnaryExpr | MulExpr '%' UnaryExpr ;
UnaryExpr   : '-' UnaryExpr | '!' UnaryExpr | PostFixExpr ;
PostFixExpr : Primary | Name ;
Primary     : Literal | '(' Expr ')' | FieldAccess | MethodCall ;
```

Each level recurses on its *own* non-terminal on the left (left-
recursion) and steps down on the right; LALR(1) handles this
efficiently.

## Statements

```
Stmt : Block | ';' | ExprStmt | BreakStmt | ReturnStmt
     | IfThenStmt | IfThenElseStmt | IfThenElseIfStmt
     | WhileStmt | ForStmt ;
```

`IfThenElseIfStmt` is split out as its own rule rather than building
`if-else-if` as nested `IfThenElseStmt`; this dodges the classic
dangling-else conflict.

## Method calls vs. instantiation

```
MethodCall: Name '(' ArgListOpt ')'
          | Name '{' ArgListOpt '}'
          | Primary '.' IDENTIFIER '(' ArgListOpt ')'
          | Primary '.' IDENTIFIER '{' ArgListOpt '}' ;
InstantiationExpr: Name '(' ArgListOpt ')' ;
```

The first `MethodCall` and `InstantiationExpr` look identical — they
are disambiguated semantically later (Ch 6/7) once symbol tables
know whether `Name` resolves to a class or a method.

## Cross-references

- [Concept: Context-free grammar](/wiki/concepts/context-free-grammar.md)
- [Concept: Compiler pipeline](/wiki/concepts/compiler-pipeline.md)
- [Tool: Yacc / Iyacc](/wiki/tools/yacc-iyacc.md)
