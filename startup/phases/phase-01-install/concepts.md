---
type: Phase Section
title: "Phase 1 — Concepts"
description: The cross-phase ideas this install phase draws on.
tags: [phase, install, concepts]
timestamp: 2026-06-27T00:00:00Z
---

# Concepts — Phase 1: Install tools

## What a toolchain is

A **toolchain** is the ordered set of programs that transform source
text into executable output. For this course:

```
source.j0
  → [JFlex]   generates a scanner
  → [BYacc]   generates a parser
  → [javac]   compiles the compiler
  → [java]    runs the compiler
  → [gcc]     assembles native output  (Ch 13 only)
```

Each tool in the chain is independently installed and versioned. A
missing or misconfigured tool breaks only the stages downstream of it.

## PATH resolution

Tools are found by searching directories listed in the `PATH`
environment variable. If `jflex --version` fails with "command not
found", `jflex` is not on your PATH — even if the binary exists
somewhere on disk.

**Diagnose:**
```bash
which jflex      # prints full path, or empty if not found
echo $PATH       # inspect the search list
```

## Version pinning vs. latest

The course was written against specific tool versions (JFlex 1.8,
BYacc 20210808) but works with any release that supports:
- JFlex: the `%int` directive and `--version` flag.
- BYacc: the `-J` (Java output) flag.

Use latest stable unless you encounter a reproducible regression.

## Related tool cards

- [JDK](../../tools/jdk.md)
- [JFlex](../../tools/jflex.md)
- [BYacc / CUP](../../tools/byacc-cup.md)
- [Make](../../tools/make.md)
- [GCC](../../tools/gcc.md)
- [Unicon](../../tools/unicon.md)
