---
type: Chapter Section
title: "Ch 3 — Overview"
description: What the chapter teaches and where its output is consumed.
resource: /ch3/
tags: [overview, lexer]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

Ch 3 introduces the very first compiler phase. You write a `.l` file
with regular expressions; Flex/Uflex turns it into a scanner that
returns `(category, lexeme)` pairs.

## Position in the pipeline

```
source.j0 ──▶ [Ch 3 lexer] ──▶ tokens ──▶ [Ch 4 parser]
```

## Outputs

- `Yylex.java` (Flex) or `javalex.icn` (Uflex), included by every
  later chapter.
- A small driver (`simple.java`, `j0.java`) that prints the token
  stream so you can sanity-check the scanner before parsing exists.

## Why it matters

Every later chapter re-uses `javalex.l`. Mistakes here cascade — a
missing keyword shows up as a parser error in Ch 4, and a missing
operator shows up as a codegen surprise in Ch 8.
