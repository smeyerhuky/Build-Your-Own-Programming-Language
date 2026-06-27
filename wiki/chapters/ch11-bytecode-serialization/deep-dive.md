---
type: Chapter Section
title: "Ch 11 — Deep dive: .j0 file format and loader"
description: Magic header search, entry-point seeding, instruction decoding.
resource: /ch11/j0machine.java
tags: [deep-dive, bytecode-format, loader]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `.j0` format & loader

## File layout

```
[ optional shebang / pre-magic bytes ]
[ "Jzero!!\0" magic ]
[ 8-byte entry-point operand ]
[ 8-byte instructions ... ]
```

## Magic search, not check

```java
byte[] magstr = "Jzero!!\0".getBytes(StandardCharsets.US_ASCII);
int i = find(magstr, code);
if (i >= 0) { codebuf = ByteBuffer.wrap(code); return true; }
```

A `find()` linear scan, not `code[0..7] == magic`. Reason: lets a
script header (`#!/usr/bin/env j0x`) precede the bytecode in the same
file.

## Boot sequence (`init`)

```java
ip = sp = 0;
if (! loadbytecode(filename)) { /* abort */ }
ip = 16;
ip = finstr = (int) (8 * getOpnd());
stack = new byte[800000];
stackbuf = ByteBuffer.wrap(stack);
```

After the magic at bytes 0–7 and the entry-point word at bytes 8–15,
the entry-point operand is multiplied by 8 to convert
"instruction count" → "byte offset."

## Operand decoding

```java
public static long getOpnd() {
   long i = 0;
   if (codebuf.get(ip+7) < 0) i = -1;
   for(int j=7; j>1; j--) i = (i<<8) | codebuf.get(ip+j);
   return i;
}
```

6-byte (48-bit) sign-extended operand. The `if (codebuf.get(ip+7) <
0) i = -1` line is the **sign-extension trick** — initialize all-ones
when the high byte is negative.

## Cross-references

- [Language: J0 bytecode](/wiki/languages/j0-bytecode.md)
- [Concept: Bytecode VM](/wiki/concepts/bytecode-vm.md)
