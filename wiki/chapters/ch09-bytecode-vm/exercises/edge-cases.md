---
type: Edge Case Catalog
title: "Ch 9 — Edge cases"
description: Common VM bugs.
resource: /ch9/j0machine.java
tags: [edge-cases, cot, vm, stack]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 9 VM

## 1. Stack imbalance per opcode

### Buggy input
A new `MUL` case that writes:

```java
case Op.MUL: {
   long val1 = stackbuf.getLong(sp);     // peek, not pop
   long val2 = stackbuf.getLong(sp - 8);
   stackbuf.putLong(sp + 8, val1 * val2);
   sp += 8;
   break;
}
```

### Symptom
A program that multiplies two values once works; multiplying twice
crashes with garbage results because `sp` grew instead of shrinking
by one slot.

### CoT trace
1. Compare to canonical `ADD`: `getLong(sp--)` twice, `putLong(sp++,
   ...)`. Net delta: `-1` slot (`sp` ends one below start).
2. The buggy `MUL` ends one slot *above* start — opposite direction.
3. After two `MUL`s the stack pointer is off by 16, every later
   load reads a stale value.

### Fix
Mirror the `ADD` shape:

```java
case Op.MUL: {
   long val1 = stackbuf.getLong(sp--);
   long val2 = stackbuf.getLong(sp--);
   stackbuf.putLong(sp++, val1 * val2);
   break;
}
```

### Lesson
Every opcode has a documented `sp` delta. Code the new opcode by
**copying a known-good sibling** and only changing the operator.

---

## 2. `bp` drift through CALL/RETURN

### Buggy input
`CALL` pushes `ip` but forgets to push `bp`:

```java
case Op.CALL: {
   push(ip);
   bp = sp;       // <-- caller's bp lost
   ip = (int)opnd;
   break;
}
```

### Symptom
Single-level calls work. Nested calls return to junk addresses
because `RETURN` restores `bp` from a value that was actually the
caller's saved `ip`.

### CoT trace
1. The call stack format is `[ ret_ip | saved_bp | locals... ]`.
2. Skipping the `saved_bp` push shifts every offset by one slot.
3. `RETURN` pops what it thinks is `bp` but is actually `ip`, then
   pops what it thinks is `ip` and gets random stack bytes.

### Fix
Push both in `CALL`, pop both in reverse in `RETURN` (see cookbook
entry 3).

### Lesson
Frame layout is a contract between `CALL`, `RETURN`, and the
compiler's `LOCAL`/`PUSH`/`POP` offsets. Any single piece getting
out of sync silently corrupts every call.

---

## 3. Operand sign-extension wrong

### Buggy input
`getOpnd()` reads 6 bytes without sign extension:

```java
public static long getOpnd() {
   long i = 0;
   for(int j=7;j>1;j--) i = (i<<8) | codebuf.get(ip+j);
   return i;
}
```

### Symptom
Negative immediates (e.g., `-1`) decode as huge positive numbers,
so `JUMP -1` becomes `JUMP 0xFFFFFFFFFF`.

### CoT trace
1. The high byte of a signed 48-bit operand carries the sign bit.
2. The canonical code special-cases negatives: `if (codebuf.get(ip+7)
   < 0) i = -1;` initializes to all-ones so the OR-shifts land
   correctly.
3. The buggy version starts from 0 and never sets the sign bits.

### Fix
Pre-initialize per the high-byte sign:

```java
long i = 0;
if (codebuf.get(ip+7) < 0) i = -1;
for(int j=7;j>1;j--) i = (i<<8) | codebuf.get(ip+j);
return i;
```

### Lesson
Sign extension is a one-line trick that's easy to forget when
porting between languages. Mismatched extension produces wildly
wrong values, not subtly wrong ones — so it shows up fast.

---

## 4. `ip` advance forgotten

### Buggy input
A new opcode case that does work but forgets `break;`:

```java
case Op.MY_OP: { do_my_work(); /* missing break */ }
case Op.NEXT_OP: { do_next_work(); break; }
```

### Symptom
Java's fall-through executes `Op.NEXT_OP` after `Op.MY_OP`, so two
instructions seem to run from one fetch.

### CoT trace
1. Java `switch` falls through unless `break`.
2. Add `break` to every case in `interp()`.

### Fix
Add `break;`. (Or use the modern `case Op.X -> { ... }` syntax which
doesn't fall through, but the rest of the file uses old-style
cases — match the surrounding style.)

### Lesson
This isn't a compiler bug — it's a Java foot-gun. Every interpreter
written in C/Java has it. Use a linter that flags missing `break`
in switch cases over enum-like constants.
