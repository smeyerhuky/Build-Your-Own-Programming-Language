---
type: Chapter Section
title: "Ch 9 — Deep dive: j0machine.java"
description: VM state, fetch, and the dispatch switch.
resource: /ch9/j0machine.java
tags: [deep-dive, vm, j0machine, interpreter]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `j0machine.java`

## State

```java
public static byte[] code, stack;
public static ByteBuffer codebuf, stackbuf;
public static int ip, sp, bp, hp, op, opr, finstr;
public static long opnd;
```

## Fetch

```java
public static void fetch() {
   op = code[ip];
   opr = code[ip+1];
   if (opr != 0) { opnd = getOpnd(); }
   ip += 8;
}
```

Every instruction is exactly 8 bytes. The mode byte `opr` decides
whether to read the operand from bytes 2–7 (`getOpnd()` reads 6
bytes, sign-extends).

## Dispatch (excerpt)

```java
case Op.ADD: {
   long val1 = stackbuf.getLong(sp--);
   long val2 = stackbuf.getLong(sp--);
   stackbuf.putLong(sp++, val1 + val2);
   break;
}
case Op.PUSH:   { push(deref(opr, opnd)); break; }
case Op.POP:    { assign(opnd, pop()); break; }
case Op.GOTO:   { ip = (int)opnd; break; }
case Op.BIF:    { if (pop() != 0) ip = (int)opnd; break; }
case Op.CALL:   { /* push ip, push bp, bp = sp, ip = opnd */ }
case Op.RETURN: { /* sp = bp, bp = pop, ip = pop */ }
case Op.HALT:   { stop("Execution complete."); break; }
```

## Boot

`init(filename)` loads the bytecode, finds the `Jzero!!\0` magic,
seeds `ip` from the entry-point word at byte 16, then allocates an
800 KB stack. The first real instruction runs from
`finstr = 8 * <entry-point operand>`.

## Cross-references

- [Concept: Bytecode VM](/wiki/concepts/bytecode-vm.md)
- [Language: J0 bytecode](/wiki/languages/j0-bytecode.md)
- [Concept: Stack frame](/wiki/concepts/stack-frame.md)
