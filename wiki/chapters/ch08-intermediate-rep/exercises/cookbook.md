---
type: Cookbook
title: "Ch 8 — Cookbook"
description: Canonical TAC lowering patterns.
resource: /ch9/tac.java
tags: [cookbook, tac, gencode]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 8 TAC

## 1. Lower an arithmetic expression

### Prompt
"Lower `a + b * c` into TAC."

### Reference solution
```
t1 = b MUL c
t2 = a ADD t1
```

In `gencode("AddExpr+")`:

```java
kids.get(0).gencode();
kids.get(1).gencode();
address dst = newTemp();
icode.add(new tac("ADD", dst, kids.get(0).addr, kids.get(1).addr));
this.addr = dst;
```

### Why this is canonical
- Children generate first, depositing their result in `kid.addr`.
- The parent emits one op using those addresses and a fresh temp.

---

## 2. Lower an `if`

### Prompt
"Lower `if (e) S1 else S2`."

### Reference solution
```
   <e>
   BIF Lt
   <S2>
   GOTO Lend
Lt: <S1>
Lend:
```

In `gencode("If")` with three kids `(cond, then, else)`:

```java
cond.gencode();
int lt = newLabel(), lend = newLabel();
icode.add(new tac("BIF", lab(lt), cond.addr));
elseBranch.gencode();
icode.add(new tac("GOTO", lab(lend)));
icode.add(new tac("LAB", lab(lt)));
thenBranch.gencode();
icode.add(new tac("LAB", lab(lend)));
```

### Why this is canonical
Two fresh labels per branching construct. `BIF` branches on
**non-zero** (truthy); the inverted layout (`else` first) makes the
common `if`-without-`else` case efficient.

---

## 3. Lower short-circuit `&&`

### Prompt
"Lower `a && b` so `b` is not evaluated when `a` is false."

### Reference solution
```
   <a>
   t = 0
   BIF Lend [inverted]    ; if !a, jump to end with t=0
   <b>
   t = b
Lend:
```

A direct way:

```java
a.gencode();
address t = newTemp();
icode.add(new tac("STORE", t, imm(0)));
int lend = newLabel();
// branch to Lend if a == 0 (i.e., if !a)
icode.add(new tac("EQ", a.addr, a.addr, imm(0)));
icode.add(new tac("BIF", lab(lend), a.addr));
b.gencode();
icode.add(new tac("STORE", t, b.addr));
icode.add(new tac("LAB", lab(lend)));
```

### Why this is canonical
Short-circuit is a **control-flow** lowering, not arithmetic. `&&`
must skip evaluating `b` when `a` is false.

---

## 4. Emit a procedure prologue/epilogue

### Prompt
"Emit TAC for a method that takes 2 ints and returns an int."

### Reference solution
```java
icode.add(new tac("proc",
                  region("local"), imm(16), imm(localsBytes)));
// ... body ...
icode.add(new tac("end"));
```

### Why this is canonical
`proc` is the explicit boundary every backend walks for; locals
count is computed from the method's symbol table.
