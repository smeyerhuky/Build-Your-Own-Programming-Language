---
type: Phase Section
title: "Phase 3 — Overview"
description: What this phase covers — the three-step Ch 3 build that produces a working token stream from hello.java.
tags: [phase, first-build, lexer, overview]
timestamp: 2026-06-27T00:00:00Z
---

# Overview — Phase 3: First build

## Position in the onboarding sequence

```
Phase 2: Clone & verify ──▶ [Phase 3: First build] ──▶ Phase 4: Extend J0
```

## What you do

1. `jflex javalex.l` — generates `Yylex.java` from the lexer spec.
2. `javac Yylex.java simple.java token.java` — compiles the scanner and its driver.
3. `java simple hello.java` — runs the driver and prints a token stream.

## Why this matters

This is the smallest meaningful end-to-end proof the toolchain works.
`hello.java` is 5 lines. The token stream it produces covers keywords,
identifiers, string literals, and punctuation — almost the entire J0
vocabulary. If this works, Ch 4 and everything that follows will
compile.

## Position in the compiler pipeline

```
hello.java (source)
   ▼
[javalex.l → Yylex.java]  ← Phase 3 builds and runs this
   ▼
token stream printed to stdout
   ▼
(Ch 4 parser would consume this next)
```

## Outputs of this phase

- `ch3/Yylex.java` compiled to `Yylex.class`.
- `ch3/simple.class`, `ch3/token.class`.
- Terminal output: one `token N: lexeme` line per token in `hello.java`.

## Time estimate

2–5 minutes once the repo is cloned.
