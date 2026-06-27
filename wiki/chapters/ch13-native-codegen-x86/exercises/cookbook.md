---
type: Cookbook
title: "Ch 13 — Cookbook"
description: Canonical x86-64 emission patterns.
resource: /ch13/x64.java
tags: [cookbook, x86-64, native]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 13 x86-64

## 1. Lower an `if` to compare + branch

### Prompt
"Emit x86-64 for `if (a < b) S1 else S2` where `a` and `b` are
locals."

### Reference solution
```
   movq   -8(%rbp), %rax        # a → rax
   cmpq   -16(%rbp), %rax       # a - b
   jl     .L1                   # branch if a < b
   <S2>
   jmp    .L2
.L1: <S1>
.L2:
```

Each instruction is one `new x64("movq", src, dst)` etc.; labels are
`new x64("lab", new x64loc("lab", id))`.

### Why this is canonical
`cmpq` then conditional jump is the standard pattern. The `jl`
(signed-less-than) opcode matches J0's signed integer semantics.

---

## 2. Emit a function prologue and epilogue

### Prompt
"Emit prologue/epilogue for a method with `N` bytes of locals."

### Reference solution
```
foo:
    pushq  %rbp
    movq   %rsp, %rbp
    subq   $N, %rsp
    ...                          # body
    movq   %rbp, %rsp
    popq   %rbp
    ret
```

As `x64` records:

```java
emit("lab", new x64loc("lab", labelForFoo));
emit("pushq", reg("%rbp"));
emit("movq",  reg("%rsp"), reg("%rbp"));
emit("subq",  imm(N),      reg("%rsp"));
// body
emit("movq",  reg("%rbp"), reg("%rsp"));
emit("popq",  reg("%rbp"));
emit("ret");
```

### Why this is canonical
SysV AMD64 prologue: save `%rbp`, set new `%rbp`, carve locals.
Epilogue mirrors it exactly.

---

## 3. Pass arguments per SysV

### Prompt
"Emit a call `bar(x, y)` where `x` and `y` are locals."

### Reference solution
```
    movq   -8(%rbp), %rdi
    movq   -16(%rbp), %rsi
    call   bar
```

### Why this is canonical
First two integer args go in `%rdi, %rsi`. The `call` instruction
pushes the return address and jumps.

---

## 4. Choose a free register

### Prompt
"Pick an unused callee-save register for a long-lived temp."

### Reference solution
```java
int r = reguse.alloc(RegUse.CALLEE_SAVE);
String name = RegUse.NAMES[r];   // e.g. "%rbx"
```

### Why this is canonical
Callee-save registers survive calls. The `RegUse` bitset tracks live
registers across the function body so temps don't clobber each
other.
