---
type: Phase Section
title: "Phase 4 — Overview"
description: What this phase covers — adding a keyword and an operator to the J0 lexer and grammar.
tags: [phase, extend-j0, keyword, operator, overview]
timestamp: 2026-06-27T00:00:00Z
---

# Overview — Phase 4: Extend J0

## Position in the onboarding sequence

```
Phase 3: First build ──▶ [Phase 4: Extend J0] ──▶ Phase 5: Full pipeline
```

## What you do

Add two small features to J0:
1. A **keyword** (`do`) in `ch3/javalex.l` and declare it in
   `ch4/j0gram.y`.
2. A **single-character operator** (`&`) in `ch3/javalex.l`.

Rebuild the lexer, rerun the driver, and confirm both features
tokenize correctly.

## Why this matters

Extending the language touches two files in two chapters. It reveals
how changes cascade: a lexer rule must pair with a grammar `%token`
declaration; the build order matters (`jflex` before `javac`). This
is the same workflow used for every real language extension in Ch 4–9.

## Outputs of this phase

- `javalex.l` extended with two new rules.
- `j0gram.y` extended with two new `%token` declarations.
- A test `.java` file that exercises both features and lexes correctly.

## Time estimate

15–30 minutes including reading the existing lexer rules for context.
