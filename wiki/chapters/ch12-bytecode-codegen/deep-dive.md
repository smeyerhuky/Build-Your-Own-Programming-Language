---
type: Chapter Section
title: "Ch 12 — Deep dive: byc.java"
description: The bytecode writer: file header, instruction stream, string table.
resource: /ch12/byc.java
tags: [deep-dive, bytecode-codegen, byc]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `byc.java`

`byc` is the bridge from in-memory `tac` records to a real `.j0`
file. Responsibilities:

1. Write the `"Jzero!!\0"` magic.
2. Write the 8-byte entry-point operand (`main`'s offset in
   instruction units).
3. Write the instruction stream as `[opcode, opr, 6-byte operand]`
   blobs.
4. Append the string table (literal contents referenced by
   `R_HEAP`-mode `PUSH` of a string address).

## Encoding loop

```java
out.write(op);                       // 1 byte
out.write(opr);                      // 1 byte
long v = operand;
for (int j = 0; j < 6; j++) { out.write((int)(v & 0xff)); v >>= 8; }
```

## String-table layout

A single `.string` region at the end of the file: each entry is a
length-prefixed UTF-8 blob. `j0.stringtab.intern(s)` returns the
offset to use as the literal's operand.

## Label back-patching

Forward jumps are emitted with operand `0`, then resolved when the
target `LAB` is reached. The compiler keeps a `Map<Integer,
List<Integer>>` from label id → list of byte offsets that need
patching.

## Cross-references

- [Language: J0 bytecode](/wiki/languages/j0-bytecode.md)
- [Concept: TAC](/wiki/concepts/three-address-code.md)
