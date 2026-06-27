---
type: Cookbook
title: "Ch 12 — Cookbook"
description: Canonical bytecode-emission patterns.
resource: /ch12/byc.java
tags: [cookbook, bytecode-codegen]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 12 bytecode codegen

## 1. Compile a method call

### Prompt
"Emit bytecode for `foo(a, b)` where `foo` is at label `Lfoo`."

### Reference solution
```
PUSH R_STACK <a-offset>
PUSH R_STACK <b-offset>
CALL R_ABS Lfoo
POP  R_STACK <result-slot>      ; if result is used
```

### Why this is canonical
- Push actuals left to right.
- `CALL` consumes the saved-`ip`/`bp` discipline by jumping to the
  callee.
- The callee leaves its return value on TOS; the caller `POP`s if
  it wants to store it.

---

## 2. Compile a `while` loop

### Prompt
"Emit bytecode for `while (e) S`."

### Reference solution
```
Ltop:  <e>
       BIF Lbody
       GOTO Lend
Lbody: <S>
       GOTO Ltop
Lend:
```

The two-label form is equivalent to "test, branch-on-false to end,
body, branch-back" — but emitted as branch-on-**true**-to-body to
match J0's `BIF`.

### Why this is canonical
Three labels per loop fits J0's `BIF`-only branch instruction.
Adding a single `BNF` (branch-on-false) opcode would simplify this
to two labels.

---

## 3. Intern a string literal

### Prompt
"Emit a `PUSH` for the string literal `"hello"`."

### Reference solution
```java
int off = j0.stringtab.intern("hello");
icode.add(new tac("PUSH", new address("string", off)));
```

`PUSH R_HEAP <off>` at byte-encode time.

### Why this is canonical
Single intern site — dedup is automatic. The string table is laid
out by `byc.java` after the instruction stream.

---

## 4. Back-patch a forward jump

### Prompt
"Emit a `BIF` whose target hasn't been emitted yet."

### Reference solution
```java
int site = icode.size();
icode.add(new tac("BIF", new address("lab", labelId)));  // operand = 0 sentinel
// ...
// When LAB labelId is reached:
icode.get(site).op1.offset = currentByteOffset;
```

### Why this is canonical
Single-pass codegen with a fix-up list, rather than two passes
(measure-then-emit).
