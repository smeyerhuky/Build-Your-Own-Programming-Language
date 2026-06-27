---
type: Tool
title: Flex / Uflex
description: Lex-style scanner generators producing Java (JFlex) or Unicon (Uflex) lexers from a .l file.
tags: [flex, jflex, uflex, lexer, scanner]
timestamp: 2026-06-27T00:00:00Z
---

# Flex / Uflex

The course uses Flex-family tools to compile `.l` files into scanners.

## .l file structure

```
%%               <- definitions section ends
<regex>  <action in target language>
...
%%               <- rules section ends
<helper code>
```

## Examples in this repo

- `/ch3/javalex.l` — full J0 lexer (used by every later chapter too).
- `/ch3/nnws.l`, `/ch3/nnws-tok.l` — toy "non-whitespace" warm-ups.

## Build artifacts

JFlex emits `Yylex.java`; Uflex emits a `.icn` file. The
chapter `makefile` re-runs the generator whenever the `.l` file
changes.

## Idioms enforced

- The `.` catch-all is always last.
- Keywords precede the `{id}` rule so they bind first.
- Whitespace and comments dispatch into helpers on the `j0` class so
  line/column tracking stays accurate.
