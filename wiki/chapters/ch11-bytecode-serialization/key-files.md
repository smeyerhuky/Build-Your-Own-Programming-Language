---
type: File Inventory
title: "Ch 11 — Key files"
description: Source files in /ch11 with their purpose.
resource: /ch11/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch11`

| File | Purpose |
|---|---|
| `j0machine.java` / `j0machine.icn` | The VM, now loading from disk via `loadbytecode()`. |
| `j0x.java` / `j0x.icn` | The standalone executor program: `j0x foo.j0`. |
| `Op.java` / `Op.icn` | 23 opcodes (HALT..STORE) + 5 register modes (R_NONE..R_HEAP). |
| `hello.java` | Test J0 source. |
| `hello.j0` | Pre-built bytecode for `hello.java`. |
| `writehello.icn` | Utility that produces a hand-rolled `hello.j0` for sanity-checking the loader. |
| `makefile` | Build glue. |
