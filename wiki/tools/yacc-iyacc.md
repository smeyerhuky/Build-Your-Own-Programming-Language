---
type: Tool
title: Yacc / Iyacc
description: Yacc-style LALR(1) parser generators producing Java or Unicon parsers from a .y file.
tags: [yacc, iyacc, parser-generator, lalr]
timestamp: 2026-06-27T00:00:00Z
---

# Yacc / Iyacc

`.y` files declare tokens, optional precedence, and production rules
with semantic actions in the target language.

## .y file structure

```
%token TOK1 TOK2 ...
%left  '+' '-'        <- optional precedence/associativity
%%
NonTerm : RHS         { semantic action $$ = ... ; }
        | OtherRHS    ;
%%
<helper code>
```

## Examples in this repo

- `/ch4/j0gram.y` — smallest complete J0 grammar.
- `/ch4/nameseq.y`, `/ch4/ns.y` — toy grammars.
- `/ch12/j0gram.y` — grammar with semantic actions that build the AST
  and feed the bytecode emitter.

## Build artifacts

In this repo, checked-in Java Yacc output includes `/ch4/Parser.java`, `/ch4/parserTokens.java`, `/ch4/parserVal.java`, plus `/ch5/parser.java` and `/ch9/parser.java` (both with `parserVal.java`); later chapters keep `j0gram.y` but don’t check in the generated Java parser sources.
Iyacc emits `j0gram.icn` / `j0gram_tab.icn` for Unicon.

## Idioms enforced

- Single-production passthroughs use `{ $$ = $1; }` to keep the AST
  smaller than the parse tree.
- Use rule-level precedence layering (`AddExpr` above `MulExpr`)
  rather than `%left` directives, so the grammar reads as a
  precedence table.
