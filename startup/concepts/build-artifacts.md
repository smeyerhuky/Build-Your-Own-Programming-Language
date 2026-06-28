---
type: Setup Concept
title: Build artifacts
description: A complete table of what each onboarding phase and compiler chapter produces and consumes.
tags: [concept, build-artifacts, generated-files]
timestamp: 2026-06-27T00:00:00Z
---

# Build artifacts

A **build artifact** is any file produced by a tool during the build.
Generated files are not committed to the repo; you produce them
locally. This page is the definitive table of what exists when, and
what produces it.

## Phase artifacts (onboarding)

| Phase | Produces | Via |
|---|---|---|
| Phase 1 | Tools on PATH | Package manager / manual install |
| Phase 2 | `ch3/Yylex.java` | `jflex javalex.l` |
| Phase 2 | `ch4/parser.java` | `byacc -J j0gram.y` |
| Phase 3 | `ch3/*.class` | `javac Yylex.java simple.java token.java` |
| Phase 4 | Updated `javalex.l` + `j0gram.y` | Hand-edited |
| Phase 5 | `ch9/*.class` → execution | `make` + `java j0 …` |

## Compiler chapter artifacts

| Chapter | Source files (committed) | Generated files (not committed) |
|---|---|---|
| Ch 3 | `javalex.l`, `simple.java`, `token.java` | `Yylex.java`, `*.class` |
| Ch 4 | `j0gram.y`, `*.java` | `parser.java`, `*.class` |
| Ch 5–9 | `tree.java`, `symtab.java`, etc. | `*.class` |
| Ch 11–12 | `Op.java`, `byc.java`, etc. | `*.class`, `hello.j0` |
| Ch 13 | `x64.java`, `x64loc.java`, etc. | `*.class`, `hello.s`, `a.out` |

## Never edit generated files

| Generated file | Why |
|---|---|
| `Yylex.java` | Overwritten every time `jflex` runs. |
| `parser.java` | Overwritten every time `byacc` runs. |
| `*.class` | Overwritten every time `javac` runs. |
| `hello.j0` | Overwritten every time the bytecode compiler runs. |
| `a.out` | Overwritten every time GCC links. |

Always edit the source (`.l`, `.y`, hand-written `.java`) and
regenerate.

## Checking for stale artifacts

If a build behaves unexpectedly after a source change:
```bash
cd chN
make clean    # remove all generated + compiled files
make          # fresh build
```

`make clean` is the nuclear option — it guarantees no stale `.class`
or generated `.java` files influence the next build.
