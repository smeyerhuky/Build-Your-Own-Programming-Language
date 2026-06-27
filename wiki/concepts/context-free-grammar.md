---
type: Concept
title: Context-free grammar
description: Production-rule description of language syntax, parsed by an LALR(1) parser generated from a Yacc-style .y file.
tags: [grammar, cfg, yacc, parser, lalr]
timestamp: 2026-06-27T00:00:00Z
---

# Context-free grammar

A **context-free grammar** (CFG) defines syntax as a set of production
rules over **non-terminals** (rule names) and **terminals** (tokens).
The course uses **Yacc**-style grammar files (`.y`) which Yacc / Iyacc
compile into an **LALR(1)** parser — bottom-up, with one-token
lookahead.

## Where it appears in the book

- First introduced in [Ch 4 — Parsing](/wiki/chapters/ch04-syntax-analysis/index.md).
- Extended chapter-by-chapter as new constructs are added.

## Anchor files

- `/ch4/j0gram.y` — the smallest complete J0 grammar.
- `/ch4/nameseq.y`, `/ch4/ns.y` — toy warm-up grammars.

## Operator precedence and associativity

J0's grammar layers precedence by rule levels:
`Expr → CondOrExpr → CondAndExpr → EqExpr → RelExpr → AddExpr → MulExpr → UnaryExpr → PostFixExpr → Primary`.
A higher level binds tighter. Associativity is encoded by which side
recursion appears on (`AddExpr '+' MulExpr` is left-associative).

## Common bugs

- **Shift/reduce conflicts** when adding a statement that can also
  appear as an expression (e.g., `if/else` without explicit
  precedence — the classic "dangling else").
- **Reduce/reduce conflicts** when two productions can reduce the
  same handle, common after collapsing two AST node types.
- Forgetting to declare a new token via `%token` causes Yacc to treat
  the symbol as a non-terminal it has no rule for.

See [`/wiki/chapters/ch04-syntax-analysis/exercises/edge-cases.md`](/wiki/chapters/ch04-syntax-analysis/exercises/edge-cases.md).
