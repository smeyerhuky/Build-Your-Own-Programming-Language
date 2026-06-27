---
type: Edge Case Catalog
title: "Ch 2 — Edge cases (design anti-patterns)"
description: Design pitfalls in early language design.
tags: [edge-cases, cot, language-design]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 2 language design

## 1. Picking a paradigm without users

### Bad prompt outcome
"We'll build a pure functional language because everyone loves
Haskell."

### Symptom
The team writes the compiler, ships it, and no one in the target
domain (say, embedded firmware engineers) uses it.

### CoT trace
1. Paradigm choice should follow the *audience*, not the *author's*
   preference.
2. Audit who will read the code. If they think in mutable state,
   give them mutable state.
3. Save paradigm experiments for personal projects, not production
   DSLs.

### Lesson
Languages live or die by their first adopters. Make the surface
familiar before optimizing the semantics for elegance.

---

## 2. Over-typed surface

### Bad prompt outcome
"Every value carries a refinement type with proofs."

### Symptom
Hello-world is 30 lines. Adoption is zero.

### CoT trace
1. Strong types pay off when the cost of being wrong is high.
2. For a small DSL, the cost is usually low and the typing system
   becomes pure friction.
3. Start with dynamic or simple-static types; tighten later.

### Lesson
Type-system ambition should match the failure cost of the domain.
J0's intentionally simple `int/double/bool/string` is a good model.

---

## 3. Modeling syntax on a personal favorite

### Bad prompt outcome
"Let's use Lisp parens everywhere."

### Symptom
Anything that's not Lisp-shaped feels foreign; tooling that
non-Lispers expect (autoindent, brace-matching) breaks.

### CoT trace
1. Syntax preference is cultural. The team's culture isn't
   necessarily the users' culture.
2. Survey the users' current tools and pick syntax compatible with
   them.
3. If your users live in VS Code with Prettier, ASCII-braces beat
   parens.

### Lesson
Syntax is a UX choice. Optimize for the people who will read it,
not the people who already love Lisp.
