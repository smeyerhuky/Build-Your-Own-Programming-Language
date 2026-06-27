---
type: Chapter Section
title: "Ch 4 — Overview"
description: What the parser phase produces and how it builds on the lexer.
resource: /ch4/
tags: [overview, parser]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

The chapter introduces context-free grammars, walks through writing
`j0gram.y`, and demonstrates Yacc generating a parser that consumes
the Ch 3 lexer's tokens.

## Position in the pipeline

```
tokens (ch3) ──▶ [Ch 4 parser] ──▶ parse tree ──▶ [Ch 5 AST]
```

## Outputs

- `Parser.java` (Yacc-generated) or `j0gram.icn` + `j0gram_tab.icn`
  (Iyacc-generated).
- A driver that reports "accepted" or surfaces syntax errors.

## Warm-up grammars

`/ch4/nameseq.y` and `/ch4/ns.y` parse sequences of names — small
enough to let you see Yacc's whole behavior on a tiny CFG before
diving into the full J0 grammar.
