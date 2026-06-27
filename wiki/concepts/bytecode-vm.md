---
type: Concept
title: Bytecode VM
description: The J0 stack-based virtual machine — fetch/decode/execute, 23 opcodes, fixed 8-byte instructions.
tags: [vm, bytecode, interpreter, stack-machine, j0machine]
timestamp: 2026-06-27T00:00:00Z
---

# Bytecode VM

The J0 VM is a stack-based interpreter. State:

- `ip` — instruction pointer (byte offset into `code`).
- `sp` — stack pointer.
- `bp` — base pointer (frame).
- `hp` — heap pointer.
- `op`, `opr`, `opnd` — last fetched opcode, register-mode byte, and operand.

## Instruction layout

Each instruction is **8 bytes**:

```
+--------+--------+-------------------------------------+
| op (1) | opr(1) | operand (6 bytes, little-endian-ish)|
+--------+--------+-------------------------------------+
```

`fetch()` reads 8 bytes starting at `ip`, then `ip += 8`. See
`/ch11/j0machine.java::fetch()`.

## Opcodes (`/ch11/Op.java`)

```
HALT=1   NOOP=2   ADD=3   SUB=4   MUL=5    DIV=6   MOD=7   NEG=8
PUSH=9   POP=10   CALL=11 RETURN=12 GOTO=13 BIF=14  LT=15   LE=16
GT=17    GE=18    EQ=19   NEQ=20   LOCAL=21 LOAD=22 STORE=23
```

## Register modes (`opr` field)

```
R_NONE=0   R_ABS=1   R_IMM=2   R_STACK=3   R_HEAP=4
```

`opr` tells the VM how to interpret the operand: literal, absolute
address in the code region, stack-relative offset, or heap offset.

## Where it appears in the book

- Built and run in [Ch 9](/wiki/chapters/ch09-bytecode-vm/index.md).
- Loaded from a `.j0` file in [Ch 11](/wiki/chapters/ch11-bytecode-serialization/index.md).

## Common bugs

- Stack imbalance — every opcode must net the right delta on `sp`. `ADD` consumes two, produces one (net -1).
- `bp` drift through `CALL`/`RETURN` — both must save and restore the caller's `bp`.
