---
type: Prompt Pattern
title: Few-shot templates (positive)
description: Reusable few-shot scaffolds that wrap cookbook entries from any chapter.
tags: [few-shot, cookbook, prompt-template, positive-examples]
timestamp: 2026-06-27T00:00:00Z
---

# Few-shot templates

Take 2–4 entries from a chapter's `exercises/cookbook.md` and slot
them into one of these scaffolds. The point of a cookbook entry is
that **the prompt → solution pair encodes a canonical pattern**, so
the LLM should generalize over the surface form.

## Template A — "Add a thing to the language"

```
You are extending the J0 compiler in this repo. Each example shows a
prompt and the canonical change. Follow the same pattern for the
final prompt.

# Example 1
Prompt: {{cookbook[0].prompt}}
Solution:
{{cookbook[0].solution}}
Why this is canonical:
{{cookbook[0].why}}

# Example 2
... (repeat) ...

# Final
Prompt: <user's new request>
Solution:
```

## Template B — "Mirror a J0 feature into a sibling language"

Use cookbook entries that show how a feature is built in both Java
and Unicon. Ask the LLM to do the same for a new feature.

## Template C — "Walk the pipeline for one feature"

Use ONE feature (say, a `for` loop) and pull the cookbook entry from
Ch 3 (lexer), Ch 4 (grammar), Ch 5 (AST), Ch 9 (codegen). The model
sees the feature touching every layer and can do the same for a new
feature like `switch`.
