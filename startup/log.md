---
type: Log
title: Startup bundle update log
description: Append-only record of generations and edits to the startup-in-a-box OKF bundle.
tags: [okf, log, startup]
timestamp: 2026-06-27T00:00:00Z
---

# Log

## 2026-06-27 — Initial startup pages

- Four quick-start pages created: `prerequisites.md`,
  `environment-setup.md`, `first-build.md`, `roadmap.md`.

## 2026-06-27 — Full OKF suite ("startup in a box")

- Bundle root `index.md` upgraded to `Startup Bundle` type with full
  child map and OKF-type table.
- Five phase folders created under `phases/`:
  - `phase-01-install` — tools installation (JDK, JFlex, BYacc, Make, GCC, Unicon).
  - `phase-02-clone-verify` — clone the repo and smoke-test every tool.
  - `phase-03-first-build` — generate lexer, compile, run against `hello.java`.
  - `phase-04-extend-j0` — add a keyword and an operator end-to-end.
  - `phase-05-full-pipeline` — run the compiler from source through VM or native binary.
- Each phase folder contains: `index.md`, `overview.md`, `checklist.md`,
  `deep-dive.md`, `concepts.md`, `exercises/{index,cookbook,edge-cases}.md`.
- Six tool quick-ref cards in `tools/`: jdk, jflex, byacc-cup, make, gcc, unicon.
- Five setup concept pages in `concepts/`: j0-language, token-stream,
  build-system, compiler-pipeline, build-artifacts.
- Three prompt-pattern pages in `prompting/`: setup-assistant,
  first-error-debug, extend-j0.
