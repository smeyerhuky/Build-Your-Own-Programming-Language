---
type: Phase
title: "Phase 3 — First build"
description: Run the full Ch 3 lexer build sequence and produce a token stream from hello.java. The simplest end-to-end proof that the toolchain works.
tags: [phase, first-build, lexer, ch3, startup]
timestamp: 2026-06-27T00:00:00Z
---

# Phase 3 — First build

**Goal:** `java simple hello.java` prints a stream of token numbers
and lexemes.

## Children

- [Overview](./overview.md)
- [Checklist](./checklist.md)
- [Concepts](./concepts.md)
- [Deep dive](./deep-dive.md)
- [Exercises](./exercises/index.md)
  - [Cookbook (positive)](./exercises/cookbook.md)
  - [Edge cases (negative)](./exercises/edge-cases.md)

## Prerequisites

Phase 2 complete — repo cloned, tools smoke-tested.

## Exit criteria

```bash
cd ch3
jflex javalex.l
javac Yylex.java simple.java token.java
java simple hello.java
# → stream of "token N: lexeme" lines, ending cleanly
```

## Key files touched in this phase

| File | Role |
|---|---|
| `ch3/javalex.l` | Flex spec → input to JFlex |
| `ch3/Yylex.java` | JFlex output → compiled by `javac` |
| `ch3/simple.java` | Driver that calls `yylex()` in a loop |
| `ch3/token.java` | Token record type |
| `ch3/hello.java` | Minimal J0 input: `public class hello { … }` |

## See also

- [First build quick-start page](../../first-build.md)
- [Token stream concept](../../concepts/token-stream.md)
- [JFlex tool card](../../tools/jflex.md)
