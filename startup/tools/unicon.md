---
type: Setup Tool
title: Unicon
description: Optional. Builds the .icn variant chapters. Ships uflex (Flex for Unicon) and iyacc (Yacc for Unicon).
tags: [tool, unicon, uflex, iyacc, optional]
timestamp: 2026-06-27T00:00:00Z
---

# Unicon

Unicon is an Icon-derived language that provides the `.icn` variants
of every chapter. The Java path is entirely self-contained; you only
need Unicon if you want to build the Unicon versions.

Unicon ships two bundled code-generation tools:
- `uflex` — the Unicon equivalent of JFlex (processes `.l` files).
- `iyacc` — the Unicon equivalent of BYacc (processes `.y` files).

See also: [Wiki: Unicon](/wiki/tools/unicon.md)

## Install

| Platform | Command |
|---|---|
| Linux (source) | `git clone https://github.com/uniconproject/unicon.git && cd unicon && ./configure && make` |
| Windows | Download installer from [unicon.sourceforge.net](http://unicon.sourceforge.net/) |

## Verify

```bash
unicon --version    # Unicon 13.x
uflex --version     # bundled with Unicon
iyacc --version     # bundled with Unicon
```

## Used by

All `.icn` chapters — same chapters as the Java path but with Unicon
source files. The output semantics are identical.

## Build pattern (Unicon path)

```bash
cd ch3
uflex javalex.l          # → javalex.icn
iyacc j0gram.y           # → parser.icn
unicon simple.icn        # compile + run
./simple hello.java      # or: unicon -x simple.icn hello.java
```

## Common errors

| Error | Fix |
|---|---|
| `uflex: command not found` | Unicon `bin/` not on PATH. Add it after install. |
| `.icn` build fails with type error | Check Unicon version. Unicon 13.1+ is required. |
