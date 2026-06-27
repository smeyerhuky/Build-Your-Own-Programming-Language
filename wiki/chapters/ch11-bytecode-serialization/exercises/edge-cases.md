---
type: Edge Case Catalog
title: "Ch 11 — Edge cases"
description: Common bytecode-format bugs.
resource: /ch11/j0machine.java
tags: [edge-cases, cot, bytecode-format, loader]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 11 file format

## 1. Magic-as-byte-0 check

### Buggy input
```java
if (Arrays.equals(Arrays.copyOf(code, 8), magstr)) { ... }
```

### Symptom
Adding a `#!/usr/bin/env j0x` shebang to the file breaks the loader
with "bad magic" — even though the magic *is* still in the file.

### CoT trace
1. Byte-0 check fails when anything precedes the magic.
2. The canonical loader uses `find()` which scans for the magic
   anywhere.
3. Restore `find()`.

### Fix
Use `find(magstr, code)` (the canonical code).

### Lesson
A file-format magic is a *delimiter*, not a *prefix*. Searching is
the safe default.

---

## 2. Endian assumption

### Buggy input
```java
for(int j = 2; j < 8; j++) i = (i<<8) | codebuf.get(ip+j);
```

(Big-endian decode of a little-endian-encoded operand.)

### Symptom
Every literal value is byte-reversed. `1` decodes as
`72057594037927936`.

### CoT trace
1. The canonical writer emits the low byte at offset 2.
2. The canonical reader iterates `j=7` down to `2`, OR-ing each
   byte left-shifted by 8 each pass — that's little-endian.
3. Iterating `j=2` to `7` is big-endian; mismatch.

### Fix
Reverse the loop direction.

### Lesson
Endianness mismatches are obvious because the values are wildly
wrong. But the *fix* must update both sides — encoder and decoder —
together.

---

## 3. Forgetting sign extension

### Buggy input
```java
public static long getOpnd() {
   long i = 0;                          // <-- always zero-extended
   for(int j=7;j>1;j--) i = (i<<8) | codebuf.get(ip+j);
   return i;
}
```

### Symptom
`-1` encoded as `0xFFFFFFFFFFFF` decodes as `281474976710655`.

### CoT trace
1. `byte` in Java is signed; `codebuf.get` returns negative for
   bytes ≥ 128.
2. `(i<<8) | b` with a negative `b` ORs in the high bits — but the
   bitmask must be sign-extended into `long`, not zero-extended.
3. Pre-initializing `i = -1` when the high byte is negative does the
   trick because the OR-mask already has the right high bits.

### Fix
Add the `if (codebuf.get(ip+7) < 0) i = -1;` line.

### Lesson
Java's signed-byte semantics combine with bitwise operators in
non-obvious ways. The fix is one line; the bug is invisible until
negative immediates appear.

---

## 4. Instruction-size assumption

### Buggy input
Someone writes a quick disassembler that does `for (int i = 0; i <
code.length; i += 4)`.

### Symptom
The disassembly is garbage after the first instruction because every
instruction is misaligned.

### CoT trace
1. J0 instructions are **8 bytes**, not 4.
2. The disassembler's step needs to be 8.

### Fix
`i += 8`.

### Lesson
Fixed-width ISAs make sense for the simplicity of the loader but
require every consumer to know the width. Document it in one
header file (`Op.java`) and reference that constant everywhere.
