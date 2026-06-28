---
type: Phase
title: "Phase 2 — Clone & verify"
description: Clone the repository and smoke-test every tool against the actual source files. Ends when `make` succeeds in at least one chapter directory.
tags: [phase, clone, verify, startup]
timestamp: 2026-06-27T00:00:00Z
---

# Phase 2 — Clone & verify

**Goal:** The repo is on disk and every tool runs successfully against
the real source files — not just test invocations.

## Children

- [Overview](./overview.md)
- [Checklist](./checklist.md)
- [Concepts](./concepts.md)
- [Deep dive](./deep-dive.md)
- [Exercises](./exercises/index.md)
  - [Cookbook (positive)](./exercises/cookbook.md)
  - [Edge cases (negative)](./exercises/edge-cases.md)

## Prerequisites

Phase 1 complete — all tools installed and version-checked.

## Exit criteria

```bash
# repo cloned
ls ch3/javalex.l          # exists

# JFlex runs on the real lexer spec
cd ch3 && jflex javalex.l && ls Yylex.java && cd ..

# BYacc / CUP runs on the real grammar
cd ch4 && byacc -J j0gram.y && ls *.java && cd ..

# Make succeeds in ch3
cd ch3 && make && cd ..
```

## Repo structure verified in this phase

```
Build-Your-Own-Programming-Language/
├── ch3/ … ch9/   ← full source chapters
├── ch11/ ch12/ ch13/
├── startup/      ← this bundle
├── wiki/         ← deep-reference OKF bundle
└── README.md
```

## See also

- [Environment setup guide](../../environment-setup.md)
- [Build artifacts concept](../../concepts/build-artifacts.md)
