---
type: Chapter Section
title: "Ch 12 — Overview"
description: End-to-end .j0 file generation from a J0 source file.
resource: /ch12/
tags: [overview, bytecode-codegen]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

`/ch12/j0` now produces `*.j0` files. New helpers:

- `byc.java` — bytecode writer (instruction encoder + magic + entry
  point + string table).
- `stringtab` — deduplicated string pool kept by `j0.java`.

## Position in the pipeline

```
typed AST ──▶ [Ch 12 gencode → byc writer] ──▶ .j0 file ──▶ [Ch 11 j0x loader]
```
