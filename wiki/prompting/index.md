---
type: Index
title: Prompting harness
description: Cross-chapter prompt patterns and catalogs that turn the per-chapter cookbook/edge-case material into reusable few-shot and chain-of-thought prompts.
tags: [okf, prompting, llm, few-shot, cot]
timestamp: 2026-06-27T00:00:00Z
---

# Prompting harness

The per-chapter `exercises/cookbook.md` and `exercises/edge-cases.md`
files are the raw material. This folder turns them into prompt
patterns.

- [Few-shot templates](./few-shot-templates.md) — for
  asking an LLM to *add* something to the compiler (a keyword, a
  statement, an opcode).
- [Chain-of-thought templates](./chain-of-thought-templates.md) —
  for asking an LLM to *debug* something the compiler does wrong.
- [Negative-example catalog](/wiki/prompting/negative-example-catalog.md) —
  cross-chapter index of edge cases, tagged so you can pull a
  themed bundle ("all shift/reduce", "all stack-frame bugs").
