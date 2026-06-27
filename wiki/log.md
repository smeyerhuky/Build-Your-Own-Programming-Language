---
type: Log
title: Bundle update log
description: Append-only record of generations and edits to this OKF bundle.
tags: [okf, log]
timestamp: 2026-06-27T00:00:00Z
---

# Log

## 2026-06-27 — Initial generation

- Bundle skeleton created at `/wiki/`.
- 10 full chapter folders (ch3–9, 11–13), each with
  `index.md`, `overview.md`, `concepts.md`, `key-files.md`,
  `deep-dive.md`, and `exercises/{cookbook,edge-cases}.md`.
- 7 book-only chapter folders (ch1, 2, 10, 14–17), each with
  `index.md`, `overview.md`, `concepts.md`, and
  `exercises/edge-cases.md`.
- Concept pages: lexical-analysis, context-free-grammar,
  abstract-syntax-tree, symbol-table, type-system,
  intermediate-representation, three-address-code, bytecode-vm,
  stack-frame, register-allocation, garbage-collection,
  compiler-pipeline.
- Language specs: J0, J0 bytecode (23 opcodes from `ch11/Op.java`),
  x86-64 surface used in `ch13/x64.java` and `ch13/x64loc.java`.
- Tool pages: Flex/Uflex, Yacc/Iyacc, Java, Unicon.
- Prompting harness: few-shot and chain-of-thought templates, plus a
  cross-chapter negative-example catalog.
