---
type: Prompt Pattern
title: Chain-of-thought templates (negative)
description: Reusable CoT scaffolds for debugging compiler bugs, built from per-chapter edge-case entries.
tags: [chain-of-thought, edge-cases, debugging, prompt-template, negative-examples]
timestamp: 2026-06-27T00:00:00Z
---

# Chain-of-thought templates

An `edge-cases.md` entry is shaped: buggy input → symptom → CoT trace
→ fix → lesson. The LLM should reproduce this *shape* on new bugs.

## Template A — "Diagnose this compiler output"

```
You are debugging the J0 compiler. Each example shows a buggy input,
the symptom, the step-by-step reasoning, the minimal fix, and the
lesson. Match the same reasoning style for the final bug.

# Example 1
Buggy input:
{{edge[0].buggy}}
Symptom:
{{edge[0].symptom}}
Reasoning:
{{edge[0].cot}}
Fix:
{{edge[0].fix}}
Lesson:
{{edge[0].lesson}}

# Example 2 ...

# Final
Buggy input:
<the new bug>
Symptom:
<observed behavior>
Reasoning:
```

## Template B — "Classify the bug first, then fix"

Useful when you have many cookbook+edge-case entries spanning
different layers. Ask the LLM to first identify *which layer* the bug
is in (lexer / parser / AST / symtab / typer / codegen / VM /
backend) and *then* descend into the fix.

## Template C — "Anti-cookbook"

Negate a cookbook entry: show the correct pattern, then ask the
model to produce the version that *misuses* it, and explain what
goes wrong. Useful for testing whether the model has internalized
the pattern.
