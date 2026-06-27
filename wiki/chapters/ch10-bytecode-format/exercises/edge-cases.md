---
type: Edge Case Catalog
title: "Ch 10 — Edge cases (IDE integration anti-patterns)"
description: Common pitfalls when wiring a lexer into an editor.
tags: [edge-cases, cot, ide, syntax-coloring]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 10 IDE integration

## 1. Re-lexing the whole file on every keystroke

### Bad prompt outcome
"On `keyup`, run `lex(buffer)`."

### Symptom
Editor jitters on large files; CPU pegged at 100%.

### CoT trace
1. Lexing is linear in file size; running it every keystroke is
   O(N) per char.
2. Most edits touch a few lines; relex *just the affected range*.
3. Need a "lex-from-position-X-with-known-start-state" API.

### Fix
Carry the lexer state at line boundaries; relex from the first
affected line, stopping when the state matches the previously
cached state.

### Lesson
Reusing the compiler's lexer in an editor demands incremental APIs
the batch compiler doesn't need.

---

## 2. Errors in highlighting that aren't compiler errors

### Bad prompt outcome
"Highlight unknown identifiers in red."

### Symptom
Every freshly typed identifier flashes red, then turns black when
the user finishes declaring it. Visually noisy.

### CoT trace
1. Highlighting decisions need to debounce on edits.
2. The lexer doesn't know about identifiers in the symbol-table
   sense; that's Ch 6's job.
3. Either restrict highlighting to lexer-known errors, or run a
   debounced parse + symtab pass for richer highlighting.

### Lesson
Highlighting and compiling have different latency budgets. Don't
bolt highlighting onto the full pipeline naïvely.
