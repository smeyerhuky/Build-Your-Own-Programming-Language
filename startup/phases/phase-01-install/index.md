---
type: Phase
title: "Phase 1 — Install tools"
description: Install every binary the J0 compiler toolchain needs. Ends when all version checks pass.
tags: [phase, install, tools, startup]
timestamp: 2026-06-27T00:00:00Z
---

# Phase 1 — Install tools

**Goal:** Every required binary is on your `PATH` and returns a
version number when asked.

## Children

- [Overview](./overview.md)
- [Checklist](./checklist.md)
- [Concepts](./concepts.md)
- [Deep dive](./deep-dive.md)
- [Exercises](./exercises/index.md)
  - [Cookbook (positive)](./exercises/cookbook.md)
  - [Edge cases (negative)](./exercises/edge-cases.md)

## Prerequisites

A computer running macOS, Linux, or Windows (with WSL recommended on
Windows). Administrator / sudo access for package installation.

## Exit criteria

Run the following and get a version on every line — no "command not
found":

```bash
java -version && javac -version
jflex --version
byacc --version          # or: cup --version
make --version
gcc --version            # optional until Phase 5
```

## Tools installed in this phase

| Tool | Required by |
|---|---|
| JDK 11+ | Every chapter (`javac`, `java`) |
| JFlex | Ch 3+ (generates `Yylex.java` from `javalex.l`) |
| BYacc -J or CUP | Ch 4+ (generates parser from `j0gram.y`) |
| GNU Make | All chapters (automates multi-step builds) |
| GCC / Clang | Ch 13 (assemble + link x86-64 output) |
| Unicon (optional) | `.icn` variant chapters |

## See also

- [Quick-start prerequisites page](../../prerequisites.md)
- [Tool cards](../../tools/index.md)
