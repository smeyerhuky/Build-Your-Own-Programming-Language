---
type: Index
title: Phases
description: Five onboarding phases, each a chapter-equivalent folder. Walk them in order from installing tools to running the full J0 pipeline.
tags: [okf, phases, index, startup]
timestamp: 2026-06-27T00:00:00Z
---

# Phases

Each phase mirrors the structure of a wiki chapter: `index.md`,
`overview.md`, `checklist.md`, `deep-dive.md`, `concepts.md`, and
`exercises/` with `cookbook.md` (positive patterns) and
`edge-cases.md` (common failures + CoT fixes).

| # | Phase | Goal | Folder |
|---|---|---|---|
| 1 | Install tools | Every required binary on your `PATH`, verified. | [`phase-01-install`](./phase-01-install/index.md) |
| 2 | Clone & verify | Repo cloned; every tool smoke-tested against the actual source. | [`phase-02-clone-verify`](./phase-02-clone-verify/index.md) |
| 3 | First build | `jflex` → `javac` → `java simple hello.java` produces a token stream. | [`phase-03-first-build`](./phase-03-first-build/index.md) |
| 4 | Extend J0 | Add a new keyword and operator end-to-end (lexer → grammar → test). | [`phase-04-extend-j0`](./phase-04-extend-j0/index.md) |
| 5 | Full pipeline | Source `.java` → TAC → VM interpretation or x86-64 native binary. | [`phase-05-full-pipeline`](./phase-05-full-pipeline/index.md) |

## Dependency order

```
Phase 1 (Install)
   ↓
Phase 2 (Clone & verify)
   ↓
Phase 3 (First build)
   ↓
Phase 4 (Extend J0)   ──┐
   ↓                    │ (can be done in parallel
Phase 5 (Full pipeline) ◄┘  if you skip Phase 4)
```

## Quick entry points

- If tools are already installed → skip to [Phase 2](./phase-02-clone-verify/index.md).
- If the repo is already cloned and tools verified → skip to [Phase 3](./phase-03-first-build/index.md).
- If you just want to extend J0 → skip to [Phase 4](./phase-04-extend-j0/index.md).
