---
type: Phase Section
title: "Phase 2 — Overview"
description: What this phase covers — cloning the repo and running each tool against the real source files.
tags: [phase, clone, verify, overview]
timestamp: 2026-06-27T00:00:00Z
---

# Overview — Phase 2: Clone & verify

## Position in the onboarding sequence

```
Phase 1: Install ──▶ [Phase 2: Clone & verify] ──▶ Phase 3: First build
```

## What you do

1. Clone the repository.
2. Inspect the chapter directory layout.
3. Run JFlex against `ch3/javalex.l` — produces `Yylex.java`.
4. Run BYacc against `ch4/j0gram.y` — produces a parser source file.
5. Run `make` in `ch3/` — confirms the makefile-based workflow works.
6. (Optional) Run `uflex ch3/javalex.l` if using the Unicon path.

## Why this matters

Phase 1 proves tools exist in isolation. Phase 2 proves they work with
*this repo's actual files*. Differences in tool versions, path
ordering, or flag spellings often surface only when the real `.l` or
`.y` file is fed in.

## Outputs of this phase

- Repository on disk at a known path.
- `ch3/Yylex.java` generated cleanly.
- `ch4/` parser source generated cleanly.
- `ch3/` build succeeds via `make`.

## Time estimate

5–10 minutes once tools are installed. Network speed for `git clone`
is the usual bottleneck.
