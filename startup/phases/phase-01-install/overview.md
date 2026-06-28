---
type: Phase Section
title: "Phase 1 — Overview"
description: What this phase covers, why each tool is needed, and what success looks like.
tags: [phase, install, overview]
timestamp: 2026-06-27T00:00:00Z
---

# Overview — Phase 1: Install tools

## Position in the onboarding sequence

```
[Phase 1: Install] ──▶ Phase 2: Clone & verify ──▶ Phase 3: First build
```

## What you do

Install six tools (JDK, JFlex, BYacc/CUP, Make, GCC, optionally
Unicon) and verify each with a version command. No source code is
touched in this phase.

## Why each tool matters

| Tool | If missing | First chapter that breaks |
|---|---|---|
| JDK | Nothing compiles or runs | Ch 3 |
| JFlex | `javalex.l` stays unprocessed; `Yylex.java` never exists | Ch 3 |
| BYacc / CUP | `j0gram.y` stays unprocessed; no parser | Ch 4 |
| Make | Manual multi-step commands instead of `make` | Ch 3 (convenience) |
| GCC / Clang | x86-64 output can't be assembled or linked | Ch 13 |
| Unicon | `.icn` variants can't be built or run | Ch 3 (Unicon path) |

## Outputs of this phase

- Every tool listed above is on your `PATH`.
- `java -version`, `jflex --version`, `byacc --version`, `make --version`
  all return a version number without errors.

## Time estimate

15–30 minutes on a clean machine. On a machine with JDK already
installed, 5–10 minutes for JFlex and BYacc alone.
