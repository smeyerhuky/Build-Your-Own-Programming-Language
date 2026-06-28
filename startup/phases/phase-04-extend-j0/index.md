---
type: Phase
title: "Phase 4 — Extend J0"
description: Add a new keyword and a new operator to J0 end-to-end — from the lexer rule through the grammar token declaration to a passing test.
tags: [phase, extend-j0, lexer, grammar, startup]
timestamp: 2026-06-27T00:00:00Z
---

# Phase 4 — Extend J0

**Goal:** A new keyword (`do`) and a new operator (`&`) exist in the
lexer and grammar, and `javalex.l` correctly tokenizes them.

## Children

- [Overview](./overview.md)
- [Checklist](./checklist.md)
- [Concepts](./concepts.md)
- [Deep dive](./deep-dive.md)
- [Exercises](./exercises/index.md)
  - [Cookbook (positive)](./exercises/cookbook.md)
  - [Edge cases (negative)](./exercises/edge-cases.md)

## Prerequisites

Phase 3 complete — you can build the lexer and produce a token stream.

## Exit criteria

After your changes:

```bash
cd ch3
# rebuild
jflex javalex.l && javac Yylex.java simple.java token.java

# test keyword
echo 'public class T { public static void main(string[] a) { do {} } }' \
  > /tmp/t.java
java simple /tmp/t.java | grep do

# test operator
echo 'public class T { public static void main(string[] a) { int x = 1 & 2; } }' \
  > /tmp/t2.java
java simple /tmp/t2.java | grep '&'
```

Both greps should return a non-empty match.

## Files changed in this phase

| File | Change |
|---|---|
| `ch3/javalex.l` | Add `"do"` keyword rule + `"&"` operator rule |
| `ch4/j0gram.y` | Add `%token DO` + `BITAND` declarations |

## See also

- [J0 language concept](../../concepts/j0-language.md)
- [Wiki Ch 3 cookbook](/wiki/chapters/ch03-lexical-analysis/exercises/cookbook.md)
- [Wiki Ch 4 cookbook](/wiki/chapters/ch04-syntax-analysis/exercises/cookbook.md)
