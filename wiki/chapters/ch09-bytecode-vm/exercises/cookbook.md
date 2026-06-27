---
type: Cookbook
title: "Ch 9 — Cookbook"
description: Canonical VM-extension patterns.
resource: /ch9/j0machine.java
tags: [cookbook, vm, opcodes]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 9 VM

## 1. Add a new opcode

### Prompt
"Add a `DUP` opcode that duplicates the top of stack."

### Reference solution
1. In `/ch9/Op.java` (and `/ch11/Op.java`), append:
   ```java
   public final static short DUP = 24;
   ```
2. In `j0machine.java::interp()`'s `switch (op)`:
   ```java
   case Op.DUP: {
      long v = stackbuf.getLong(sp);   // peek, don't pop
      push(v);
      break;
   }
   ```
3. In `Op.icn` (Unicon), mirror the constant.

### Why this is canonical
- Op number must be unique and added in the same place each chapter
  (file-level constants).
- New cases in `interp` follow the same shape: read state, mutate,
  break.

---

## 2. Add a stdout-write opcode

### Prompt
"Add `WRITELN` that prints the top of stack as an integer and a
newline."

### Reference solution
```java
case Op.WRITELN: {
   long v = pop();
   System.out.println(v);
   break;
}
```

### Why this is canonical
I/O lives inside the VM as a single opcode; the compiler emits
`WRITELN` rather than a sequence of CALL into a runtime library
(which would need a calling-convention contract).

---

## 3. Save/restore registers across CALL

### Prompt
"Make `CALL` and `RETURN` correctly save and restore `ip` and `bp`."

### Reference solution
```java
case Op.CALL: {
   push(ip);          // return address
   push(bp);          // caller frame
   bp = sp;
   ip = (int)opnd;
   break;
}
case Op.RETURN: {
   sp = bp;
   bp = (int)pop();
   ip = (int)pop();
   break;
}
```

### Why this is canonical
`CALL` is the only place that *creates* a new frame; `RETURN` is the
only place that destroys one. Keeping the save/restore localized
prevents drift.

---

## 4. Halt on bad opcode

### Prompt
"Stop cleanly on an unknown opcode rather than running off the end."

### Reference solution
After the `switch` body, add a `default`:

```java
default: stop("bad opcode " + op + " at ip=" + (ip-8));
```

### Why this is canonical
Defensive default with the *previous* `ip` (since `fetch` already
advanced it by 8) makes diagnostics point at the offending byte.
