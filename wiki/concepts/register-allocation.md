---
type: Concept
title: Register allocation
description: Mapping virtual registers (temps) onto a finite set of physical x86-64 registers, with spilling when needed.
tags: [register-allocation, spilling, x86-64, reguse]
timestamp: 2026-06-27T00:00:00Z
---

# Register allocation

The Ch 13 backend uses a small allocator over the integer GP registers
of x86-64. The allocator tracks which registers are live and picks a
free one for each temp; if none is free, it spills the oldest temp to
the stack.

## Anchor files

- `/ch13/RegUse.java` — bitset of live registers and helpers.
- `/ch13/reguse.icn` — the same in Unicon.
- `/ch13/x64loc.java` — represents the chosen register or memory slot
  (modes 1–6: register, integer literal, displacement(reg), label(reg),
  immediate `$N`, label).

## Calling convention used by the backend

The course follows SysV AMD64:

- Args 1–6 in `%rdi, %rsi, %rdx, %rcx, %r8, %r9`.
- Return value in `%rax`.
- Caller-save: `%rax`, `%rcx`, `%rdx`, `%rsi`, `%rdi`, `%r8`–`%r11`.
- Callee-save: `%rbx`, `%rbp`, `%r12`–`%r15`.

## Common bugs

- Using a caller-save register across a `CALL` without spilling.
- Forgetting to free a register when its live range ends, slowly
  starving the allocator into spills.
