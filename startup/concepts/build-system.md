---
type: Setup Concept
title: Build system
description: How jflex, byacc, javac, and make fit together to produce the J0 compiler from source.
tags: [concept, build-system, make, jflex, byacc, javac]
timestamp: 2026-06-27T00:00:00Z
---

# Build system

The J0 compiler build uses four independent tools chained together.
Understanding their roles and dependencies prevents the most common
"why won't it compile?" frustrations.

## The four tools

```
javalex.l ─[jflex]──▶ Yylex.java ─┐
j0gram.y  ─[byacc]──▶ parser.java  ├─[javac]──▶ *.class ─[java]──▶ run
*.java (hand-written) ─────────────┘
```

| Tool | Input | Output | When to re-run |
|---|---|---|---|
| `jflex` | `javalex.l` | `Yylex.java` | Whenever `javalex.l` changes |
| `byacc -J` | `j0gram.y` | `parser.java` | Whenever `j0gram.y` changes |
| `javac` | `*.java` | `*.class` | Whenever any `.java` changes |
| `java` | `*.class` | (runs the program) | On demand |

## Critical dependency order

1. `byacc` must run before `jflex` (because `Yylex.java` imports
   `parser.DO`, `parser.CLASS`, etc.).
2. `jflex` must run before `javac` (because `simple.java` references
   `Yylex`).
3. `javac` must run before `java`.

`make` encodes these dependencies in the chapter's `makefile` so you
can just run `make` and trust the order.

## What `make` does

```makefile
Yylex.java: javalex.l parser.java
    jflex javalex.l

parser.java: j0gram.y
    byacc -J j0gram.y

%.class: %.java Yylex.java parser.java
    javac $<
```

If only `javalex.l` changed, `make` re-runs `jflex` and then
recompiles affected `.java` files — it skips `byacc` because
`j0gram.y` is unchanged.

## See also

- [Make tool card](../tools/make.md)
- [Build artifacts concept](./build-artifacts.md)
