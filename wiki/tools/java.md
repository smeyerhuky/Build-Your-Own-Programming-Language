---
type: Tool
title: Java
description: Primary implementation language. The course code is written in straightforward, pre-generics-heavy Java.
tags: [java, jvm]
timestamp: 2026-06-27T00:00:00Z
---

# Java

Every chapter ships a Java implementation alongside the Unicon one.
The code style is intentionally simple — package declarations, plain
classes, public static fields for opcodes, `System.out.println` for
output — to keep the focus on compiler concepts.

## Build

Each chapter has a `makefile` with a Java target that runs
`javac *.java` after the lexer/parser have been regenerated.

## Notable type uses

- `HashMap<String,symtab_entry>` in `/ch6/symtab.java`.
- `ByteBuffer` over a `byte[]` in `/ch11/j0machine.java` for raw
  bytecode access.
- `ArrayList` (in `tree.java`) for n-ary children.
