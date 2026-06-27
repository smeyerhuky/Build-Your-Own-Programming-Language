---
type: Chapter Section
title: "Ch 11 — Overview"
description: From in-memory bytecode to .j0 files and back.
resource: /ch11/
tags: [overview, bytecode-format, serialization]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

The `.j0` file format is dirt simple: an ASCII magic header followed
by a stream of 8-byte instructions. The loader **searches** for the
magic — that's deliberate, so a shell shebang can precede it.

## Position in the pipeline

```
TAC (ch8/9) ──▶ [bytecode emit] ──▶ .j0 file ──▶ [Ch 11 j0x loader] ──▶ stdout
```

## Programs

- `j0x` — load + run a `.j0` file.
- `writehello.icn` — utility that writes a hand-rolled `hello.j0`.

## Files in the wild

- `hello.j0` — bytecode of `hello.java`.
- `hello.java` — the source.
