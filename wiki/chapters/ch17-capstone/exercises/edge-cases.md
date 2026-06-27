---
type: Edge Case Catalog
title: "Ch 17 — Edge cases (capstone pitfalls)"
description: Common mistakes when picking and scoping a capstone.
tags: [edge-cases, cot, capstone, scope]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 17 capstones

## 1. Picking a feature that requires rewriting the runtime

### Bad outcome
"Add real threads."

### Symptom
Two weeks in, the entire VM is being rewritten and the original
chapters' contracts have all changed.

### CoT trace
1. Threading is cross-cutting — VM, stack frames, GC, runtime
   library, FFI all change.
2. The capstone scope should fit the layers you've already built.
3. Pick something *additive*: a new statement, a new type, a new
   pass.

### Lesson
"Easy from the user side" doesn't mean "easy from the implementer
side." Threads, GC, and exceptions are all cross-cutting; generics
and a new operator are not.

---

## 2. Skipping the front-end for "the interesting part"

### Bad outcome
"I'll just add codegen for exceptions; I can hand-edit the AST
files."

### Symptom
You can demo the codegen but no real J0 program can use the
feature; tests are brittle synthetic inputs.

### CoT trace
1. End-to-end ownership matters — without lexer + grammar + AST +
   types, the feature is only half-real.
2. Plan the capstone as a tour through every chapter.
3. Time-box each layer; finish one before the next.

### Lesson
A capstone is valuable because it exercises the *whole* compiler.
Skipping layers turns it back into a one-chapter exercise.

---

## 3. No tests for the capstone

### Bad outcome
"I'll write tests after."

### Symptom
The capstone ships with no regression coverage; the next chapter's
changes silently break it.

### CoT trace
1. New features without tests rot.
2. Add one end-to-end test per cookbook entry you write.
3. Add one end-to-end *bad-input* test per edge case.

### Lesson
The cookbook and edge-cases sections of this wiki are also a test
plan. Encode each entry as a runnable case.
