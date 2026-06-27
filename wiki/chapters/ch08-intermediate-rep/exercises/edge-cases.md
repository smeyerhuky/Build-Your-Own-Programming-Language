---
type: Edge Case Catalog
title: "Ch 8 — Edge cases"
description: Common IR-generation bugs.
resource: /ch9/tac.java
tags: [edge-cases, cot, tac, gencode]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 8 TAC

## 1. Temp aliasing

### Buggy input
A naive temp allocator that reuses one shared register:

```java
address newTemp() { return TEMP_R0; }
```

### Symptom
`a + b * c` computes the right type but produces wrong values:
`MUL` clobbers `TEMP_R0`, then `ADD` reads `TEMP_R0` and gets `b*c`
twice.

### CoT trace
1. Subexpressions need their own destination slots.
2. Reusing one temp creates **aliasing** — sibling subexpressions
   overwrite each other's results.
3. Fix by minting a fresh temp per node.

### Fix
```java
address newTemp() { return new address("local", nextOffset += 8); }
```

### Lesson
Temps are cheap (8 bytes each). Reuse can come later as an
optimization (live-range analysis); never start without it.

---

## 2. Straight-line `&&`

### Buggy input
Lowering `a && b` as `AND a, b, t`.

### Symptom
`obj != null && obj.field == 0` crashes when `obj` is null —
`obj.field` was evaluated even though the LHS was false.

### CoT trace
1. `&&` is supposed to short-circuit.
2. A straight-line `AND` evaluates *both* operands.
3. Lower as control flow instead (see cookbook entry 3).

### Fix
Replace the straight-line `AND` with the BIF/GOTO pattern.

### Lesson
Boolean operators in source languages are *not* equivalent to
bitwise/arithmetic ops; the lowering must preserve evaluation order.

---

## 3. Missing `proc` / `end`

### Buggy input
A `gencode("MethodDecl")` that emits the body's TAC but forgets to
bracket it with `proc` / `end`.

### Symptom
Bytecode emission can't find function boundaries; calls jump into
the middle of arbitrary procedures.

### CoT trace
1. Backends rely on `proc`/`end` as procedure delimiters.
2. Without them, every backend has to re-derive boundaries from
   labels or method symbols.
3. Add the brackets in `gencode("MethodDecl")`.

### Fix
```java
icode.add(new tac("proc", region("local"), imm(parmBytes), imm(localsBytes)));
body.gencode();
icode.add(new tac("end"));
```

### Lesson
Conventions are part of the IR contract. Don't push convention into
the backends — encode it in the IR.

---

## 4. Address with wrong region

### Buggy input
A temp emitted with `region = "global"` instead of `"local"`.

### Symptom
The VM's `PUSH/POP` paths interpret the operand as an absolute code
address; reading/writing it corrupts the code segment.

### CoT trace
1. The `address.region` field tells the backend (and the VM) *how to
   interpret* the offset.
2. A local temp is stack-relative, not absolute.
3. The temp constructor must set `region = "local"` and the offset
   must be relative to `bp`.

### Fix
Always construct temps through one `newLocal(int bytes)` helper that
sets the region correctly.

### Lesson
Bugs in operand region are silent: the VM happily reads from
wherever you point it. Centralize address construction.
