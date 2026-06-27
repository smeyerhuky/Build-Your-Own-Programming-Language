---
type: Chapter Section
title: "Ch 13 — Deep dive: x64.java & x64loc.java"
description: How instructions and operands are represented and printed in AT&T syntax.
resource: /ch13/x64.java
tags: [deep-dive, x86-64, x64, x64loc, att-syntax]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `x64.java` & `x64loc.java`

## `x64` — one instruction

```java
class x64 {
   String op;
   x64loc opnd1, opnd2;
   // op opnd1, opnd2  (two-operand)
   // op opnd1         (one-operand)
   // op               (zero-operand)
   // op == "lab"      → "<label>:"
}
```

Printing logic (`/ch13/x64.java`):

```java
if (op.equals("lab"))
   f.println(opnd1.str() + ":");
else {
   f.print("\t" + op);
   if (opnd1 != null) f.print(" " + opnd1.str());
   if (opnd2 != null) f.print("," + opnd2.str());
   f.println();
}
```

The leading tab indents non-label lines, matching `as` conventions.

## `x64loc` — one operand, 6 modes

| `mode` | Constructor | Prints as |
|---|---|---|
| 1 | `x64loc(String reg)` | `reg` (e.g. `%rax`) |
| 2 | `x64loc(int i)` | the integer literally |
| 3 | `x64loc("reg", int off)` | `off(reg)` |
| 4 | `x64loc("reg", String lbl)` | `lbl(reg)` |
| 5 | `x64loc("imm", int n)` | `$n` |
| 6 | `x64loc("lab", int n)` | `.Ln` (or the string for non-integer offsets) |

The constructor for `mode 3/5/6` dispatches on the `reg` string:

```java
public x64loc(String r, int off) {
   if (r.equals("imm")) { offset = off; mode = 5; }
   else if (r.equals("lab")) { offset = off; mode = 6; }
   else { reg = r; offset = off; mode = 3; }
}
```

## Calling convention enforced

`j0.java` (Ch 13) emits SysV AMD64 prologue/epilogue per method and
funnels parameters through `%rdi, %rsi, %rdx, %rcx, %r8, %r9`,
spilling further args via `pushq`.

## Cross-references

- [Language: x86-64](/wiki/languages/x86-64.md)
- [Concept: Register allocation](/wiki/concepts/register-allocation.md)
- [Concept: Stack frame](/wiki/concepts/stack-frame.md)
