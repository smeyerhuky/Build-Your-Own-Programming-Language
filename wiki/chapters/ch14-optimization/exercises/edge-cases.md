---
type: Edge Case Catalog
title: "Ch 14 — Edge cases (transpiler anti-patterns)"
description: Common pitfalls in source-to-source translation.
tags: [edge-cases, cot, transpiler]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 14 transpilers

## 1. Macro that captures user variables

### Bad input
```
#define swap(a,b) { int tmp = a; a = b; b = tmp; }
swap(x, tmp);   // user's local also called `tmp`
```

### Symptom
`tmp` inside the macro shadows the user's `tmp`; behavior depends
on which one wins.

### CoT trace
1. C-style macros are textual substitution — no scope, no hygiene.
2. The user's variable name collides with the macro's local.
3. Either gensym every macro local (`_tmp_42`) or switch to
   hygienic macros.

### Lesson
"Just text substitution" macros are simple but unsafe. The price of
hygiene is an AST-based macro system.

---

## 2. Transpiler losing source-map info

### Bad outcome
The transpiler emits the target language but no source map; users'
errors point at lines in the generated code, not the source.

### CoT trace
1. The transpiler walks the AST and emits text — each emitted line
   knows what AST node it came from.
2. Record per-line `(srcFile, srcLine)` alongside the emitted text.
3. Emit a source-map blob (JSON sourcemap v3 is the standard).

### Lesson
Source maps are not optional for production transpilers. Build them
into the emit walk from day one.
