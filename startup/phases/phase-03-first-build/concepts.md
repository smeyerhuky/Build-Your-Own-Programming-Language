---
type: Phase Section
title: "Phase 3 — Concepts"
description: The cross-phase ideas the first-build phase draws on.
tags: [phase, first-build, concepts, tokens, lexer]
timestamp: 2026-06-27T00:00:00Z
---

# Concepts — Phase 3: First build

## What a token is

A **token** is a `(category, lexeme)` pair. The lexer collapses
runs of characters into tokens; the parser sees only categories.

```
Source text:   int  x  =  42  ;
Token stream:  INT  ID  =  INT_LIT  SEMI
               ^    ^        ^
               category; lexeme is the original text
```

See [Token stream concept](../../concepts/token-stream.md) for more.

## How JFlex works

JFlex reads a `.l` file with three `%%`-separated sections:

1. **Definitions** — macros like `id = ([a-zA-Z_][a-zA-Z0-9_]*)`.
2. **Rules** — `<regex>  { <Java action> }` pairs.
3. **Code** — helper methods appended to the generated class.

It builds an NFA from the regexes, converts it to a DFA, and emits
`Yylex.java`. At runtime, `Yylex.yylex()` reads the next token.

## Rule ordering in javalex.l

1. Comments and whitespace first (no token returned).
2. Keywords before the `{id}` rule (so `"do"` wins over `{id}` for input `do`).
3. Longer operators before their single-character prefixes where needed.
4. The `{id}` catch-all.
5. Literal/numeric rules.
6. The `.` catch-all last (triggers a lex error).

## Related concept pages

- [Token stream](../../concepts/token-stream.md)
- [Compiler pipeline](../../concepts/compiler-pipeline.md)
- [JFlex tool card](../../tools/jflex.md)
- [Wiki: Lexical analysis](/wiki/concepts/lexical-analysis.md)
