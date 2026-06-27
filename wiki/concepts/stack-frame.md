---
type: Concept
title: Stack frame
description: Per-call activation record holding parameters, locals, return address, and saved base pointer.
tags: [stack-frame, activation-record, call, return]
timestamp: 2026-06-27T00:00:00Z
---

# Stack frame

A **stack frame** (activation record) holds everything a single
procedure call needs:

- Inbound parameters.
- Saved return address (where to jump on `RETURN`).
- Saved caller `bp` (so we can restore it).
- Local variables.
- Caller-save / callee-save registers (in the x86-64 backend).

## Layout in J0's VM

```
high addr
 |  parameter n
 |  ...
 |  parameter 1     <-- bp + 16
 |  saved bp        <-- bp + 8
 |  return addr     <-- bp
 |  local 1         <-- bp - 8
 |  ...
 |  local m         <-- sp
low addr
```

Exact offsets follow `/ch9/j0machine.java`'s `CALL` and `RETURN`
handlers.

## In x86-64 (Ch 13)

`/ch13/j0.java` emits a prologue that pushes `%rbp`, sets `%rbp =
%rsp`, then subtracts the locals area from `%rsp`. The epilogue
reverses this. Callee-save registers `%rbx`, `%r12`–`%r15` are saved
into the frame; caller-save registers (`%rax`, `%rcx`, `%rdx`,
`%rsi`, `%rdi`, `%r8`–`%r11`) live across the call only if the caller
spills them.

## Common bugs

- Allocating locals before saving `bp` clobbers the caller's frame
  pointer.
- Off-by-eight on operand offsets is the most common single bug — every
  J0 slot is 8 bytes.
