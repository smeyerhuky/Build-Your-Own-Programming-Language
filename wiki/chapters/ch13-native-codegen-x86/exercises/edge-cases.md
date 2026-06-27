---
type: Edge Case Catalog
title: "Ch 13 — Edge cases"
description: Common native-codegen bugs.
resource: /ch13/x64.java
tags: [edge-cases, cot, x86-64, register-allocation]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 13 x86-64

## 1. Caller-save register live across `call`

### Buggy input
Temp `t` lands in `%rax`; codegen emits

```
    movq   $5, %rax       # t = 5
    call   foo            # %rax clobbered by foo's return value
    addq   %rax, %rcx     # uses stale t
```

### Symptom
The program prints garbage. `t` was overwritten by `foo`'s return.

### CoT trace
1. `%rax` is **caller-save**: the callee may clobber it freely.
2. If a temp's live range crosses a `call`, it must live in a
   callee-save register or be spilled.
3. The allocator must track live-across-call and avoid
   `%rax/%rcx/%rdx/%rsi/%rdi/%r8..%r11` for such temps.

### Fix
Pick `%rbx` / `%r12` / `%r13` / `%r14` / `%r15` for long-lived
temps; or `pushq %rax` / `popq %rax` to spill.

### Lesson
Calling convention isn't optional. The allocator must know which
registers survive a `call` and pick accordingly.

---

## 2. Forgetting to restore callee-save

### Buggy input
A method uses `%rbx` but doesn't save it on entry.

### Symptom
The caller's `%rbx` value is silently overwritten; the caller
crashes much later, far from the actual bug.

### CoT trace
1. SysV says callee-save registers must hold the same value at
   return as at entry.
2. The callee must `pushq %rbx` in the prologue and `popq %rbx` in
   the epilogue.
3. The allocator must track which callee-saves it touches.

### Fix
Emit `pushq %rbx` after the prologue's `movq %rsp, %rbp` and
matching `popq %rbx` before the epilogue's `movq %rbp, %rsp`.

### Lesson
"Crashes far from the cause" is the hallmark of an ABI bug. Always
audit which callee-saves your code uses.

---

## 3. AT&T operand order swap

### Buggy input
A `movq` emitted as `movq %rax, $5` instead of `movq $5, %rax`.

### Symptom
`as` reports "operand size mismatch for `mov`" — or, worse, the
assembler accepts it and the runtime behavior is wrong because the
destination operand was actually treated as the source.

### CoT trace
1. AT&T syntax is **`op src, dst`**.
2. Intel syntax is the reverse — easy to confuse if you context-
   switch.
3. The fix is to swap operand order at the `new x64` call site.

### Fix
`new x64("movq", imm(5), reg("%rax"))`.

### Lesson
The order convention lives in *one place* (the emitter helper).
Wrap your emission so the call site reads naturally:
`mov(dst, src)` or `mov(src, dst)` — pick one and stay consistent.

---

## 4. Frame size not 16-byte aligned at call

### Buggy input
Locals total 24 bytes; the prologue does `subq $24, %rsp`.

### Symptom
The function works alone; calling certain libc functions
(especially anything using SSE) crashes with `SIGSEGV` inside libc.

### CoT trace
1. SysV requires `%rsp` to be 16-byte aligned at the `call`
   instruction.
2. `call` pushes 8 bytes (the return address), so on entry the
   callee sees `%rsp` 8-mod-16.
3. The callee must allocate locals such that after the `subq` (and
   any pushes), `%rsp` is back to 16-mod-16 before the next `call`.
4. 24 → round up to 32.

### Fix
`subq $32, %rsp` instead of `$24`.

### Lesson
The 16-byte alignment requirement is invisible until you call into
libc/SSE code. Round the locals area to 16-byte multiples
unconditionally.
