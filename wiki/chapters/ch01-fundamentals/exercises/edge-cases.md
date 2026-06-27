---
type: Edge Case Catalog
title: "Ch 1 — Edge cases (anti-patterns)"
description: Design anti-patterns when motivating a new language.
tags: [edge-cases, cot, motivation]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 1 motivation

## 1. Building a language when a library would do

### Bad prompt outcome
"We need a config DSL with conditionals."

### Symptom
Six months later the DSL has variables, loops, modules, and a
debugger — and it's strictly worse than the host language.

### CoT trace
1. The DSL grew because the team kept needing the next host feature.
2. A new language is justified by *not needing those features*;
   needing them is the signal to use the host language.
3. Refactor as a host-language library with declarative builders.

### Lesson
Reach for a DSL when the *grammar* of the domain wants to be
visible. Reach for a library when only the *operations* do.

---

## 2. Inventing syntax for the sake of it

### Bad prompt outcome
"Let's use Greek letters for our math DSL."

### Symptom
Editors can't render it, code review reads like cryptic poetry,
nobody contributes.

### CoT trace
1. Syntax is a tax on every reader and every tool.
2. Stick with ASCII unless the domain *demands* otherwise (e.g.,
   APL successors).
3. Use familiar shapes (`{ }`, `( )`, `;`) so editors already
   highlight your language.

### Lesson
Novelty in syntax is rarely worth the tooling cost. Save weirdness
for semantics.
