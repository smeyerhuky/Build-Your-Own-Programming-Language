---
type: Phase Section
title: "Phase 2 — Concepts"
description: The cross-phase ideas this clone-and-verify phase draws on.
tags: [phase, clone, verify, concepts]
timestamp: 2026-06-27T00:00:00Z
---

# Concepts — Phase 2: Clone & verify

## The repository as a build unit

Each `ch*/` folder is an independent project. It ships a `makefile`
that encodes the correct build order for its chapter. The chapters do
*not* share a top-level build system — you `cd` into a chapter
directory and run `make` there.

## Generated files vs. checked-in files

| File | Origin |
|---|---|
| `javalex.l` | Checked in — hand-written Flex spec |
| `j0gram.y` | Checked in — hand-written Yacc grammar |
| `Yylex.java` | **Generated** by JFlex from `javalex.l` |
| parser source | **Generated** by BYacc/CUP from `j0gram.y` |
| `*.class` | **Compiled** by `javac` |

Never edit generated files (`Yylex.java`, parser `.java`). Your
changes will be overwritten the next time `jflex` or `byacc` runs.
Always edit the `.l` or `.y` source and regenerate.

## Build artifacts concept

See [Build artifacts](../../concepts/build-artifacts.md) for a full
table of what each phase produces and consumes.

## Related tool cards

- [JFlex](../../tools/jflex.md)
- [BYacc / CUP](../../tools/byacc-cup.md)
- [Make](../../tools/make.md)
