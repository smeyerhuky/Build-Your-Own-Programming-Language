---
type: Edge Case Catalog
title: "Ch 16 — Edge cases (production-language pitfalls)"
description: Anti-patterns when going from toy compiler to shippable language.
tags: [edge-cases, cot, language-design, production]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 16 production languages

## 1. Standard library written too early

### Bad outcome
The team ships v1.0 with a hand-rolled stdlib that they then can't
deprecate without breaking every user.

### CoT trace
1. Early users build on whatever's there.
2. The stdlib calcifies before patterns settle.
3. Ship v1.0 with a *minimal* stdlib and a clear "experimental"
   marker on extensions.

### Lesson
The stdlib is forever. Push library code into community packages
until usage proves a pattern belongs in core.

---

## 2. No deprecation policy

### Bad outcome
Breaking changes appear in minor releases. Users stop upgrading.

### CoT trace
1. Without a policy, every release is high-risk for users.
2. Semantic versioning + a multi-release deprecation window (`x.y`
   warns, `x.y+1` errors, `x.y+2` removes) restores trust.
3. Write the policy *before* the second user shows up.

### Lesson
Compatibility is a UX feature, not a technical limitation.

---

## 3. Premature performance focus

### Bad outcome
The team spends six months on a JIT before the language has 10
users.

### CoT trace
1. The hardest problem before product-market fit is "does anyone
   want this?", not "is it fast?"
2. A clean interpreter beats a buggy JIT for adoption.
3. Optimize only once the syntax / semantics are stable.

### Lesson
Speed comes after shape. Most "real" languages were slow for years
before they were fast.
