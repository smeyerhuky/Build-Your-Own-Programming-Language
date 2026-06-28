---
type: Phase Section
title: "Phase 5 — Deep dive: the two back-ends compared"
description: Side-by-side look at the VM interpreter path (Ch 9) and the x86-64 native path (Ch 13), including where they diverge from the shared front-end.
tags: [phase, full-pipeline, deep-dive, vm, native, backend]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive — Phase 5: The two back-ends compared

## Where the front-end ends and back-ends begin

Chapters 3–8 are the **shared front-end**. Every back-end starts from
the same Ch 8 output: a list of three-address code (TAC) instructions
annotated with type and symbol-table information.

```
Ch 8 output (TAC)
   ├──▶ Ch 9   j0machine.java   direct TAC execution
   ├──▶ Ch 11  bytecode serial  TAC → .j0 binary file
   ├──▶ Ch 12  byc.java         AST → bytecode (alternative to Ch 11)
   └──▶ Ch 13  x64.java         AST → x86-64 assembly
```

## Ch 9: Stack-VM interpreter

`j0machine.java` walks the TAC list and executes each instruction
against an operand stack and a variable store. There is no compilation
step — execution is purely interpretive.

**Advantages:** simple to build, easy to debug (add a print per
opcode), no native toolchain needed.

**Disadvantages:** slow (interpreted), not portable across instruction
sets.

## Ch 13: x86-64 native code generator

`x64.java` walks the **AST** (not TAC) and emits AT&T-syntax x86-64
assembly text. `x64loc.java` tracks register assignments. The output
is assembled and linked by the host GCC/Clang.

**Key design decisions:**
- Calling convention: System V AMD64 ABI (Linux/macOS x86-64).
- String representation: null-terminated C strings (so `printf` works).
- Register allocation: simple spill-everything approach (no graph
  coloring at this stage).

**Apple Silicon note:** M-series Macs are arm64, not x86-64. Ch 13
output cannot run natively on them without Rosetta or an x86 VM.

## Bytecode path (Ch 11–12)

The `.j0` bytecode format uses an 8-byte fixed-width opcode encoding
with a magic header `Jzero!!\0`. There are 23 opcodes (see
`ch11/Op.java`). The bytecode interpreter in Ch 11 executes the
`.j0` file; Ch 12 generates the `.j0` file from the AST.

This is the middle ground: portable like the VM, more compact than TAC.

## Cross-references

- [Wiki: Bytecode VM](/wiki/concepts/bytecode-vm.md)
- [Wiki: Stack frame](/wiki/concepts/stack-frame.md)
- [Wiki: Register allocation](/wiki/concepts/register-allocation.md)
- [Wiki: J0 bytecode spec](/wiki/languages/j0-bytecode.md)
- [Wiki: x86-64 surface](/wiki/languages/x86-64.md)
