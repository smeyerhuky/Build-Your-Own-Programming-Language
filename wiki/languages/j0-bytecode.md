---
type: Language Spec
title: J0 bytecode
description: 23-opcode stack-machine ISA executed by j0machine; serialized in a .j0 file with the "Jzero!!\0" magic header.
resource: /ch11/Op.java
tags: [j0-bytecode, isa, stack-machine, bytecode-format]
timestamp: 2026-06-27T00:00:00Z
---

# J0 bytecode

A `.j0` file is a flat stream of 8-byte instructions prefixed by a
magic header. The interpreter (`j0machine.java`) is a stack machine
with a heap region for objects.

## File header

- Bytes 0–7: ASCII `Jzero!!\0` magic. Searched for at load time;
  located via `find()` so a self-execution shebang can precede it.
- Bytes 8–15: entry-point operand (giving the byte offset of the first
  instruction in instruction-units; the loader multiplies by 8).

## Instruction layout

```
byte 0 : opcode (Op.*)
byte 1 : register mode (R_NONE | R_ABS | R_IMM | R_STACK | R_HEAP)
bytes 2-7 : 48-bit operand (sign-extended)
```

Every instruction is exactly 8 bytes — `fetch()` does `ip += 8`.

## Opcodes (from `/ch11/Op.java`)

| # | Mnemonic | Meaning |
|---|---|---|
| 1  | `HALT`   | Stop execution (prints "Execution complete.") |
| 2  | `NOOP`   | Do nothing |
| 3  | `ADD`    | Pop b, pop a, push a+b |
| 4  | `SUB`    | Pop b, pop a, push a-b |
| 5  | `MUL`    | Pop b, pop a, push a*b |
| 6  | `DIV`    | Pop b, pop a, push a/b |
| 7  | `MOD`    | Pop b, pop a, push a%b |
| 8  | `NEG`    | Pop a, push -a |
| 9  | `PUSH`   | Push operand (deref'd via `opr` mode) |
| 10 | `POP`    | Pop top, store at operand |
| 11 | `CALL`   | Call procedure at operand |
| 12 | `RETURN` | Return from procedure |
| 13 | `GOTO`   | ip := operand |
| 14 | `BIF`    | Pop; if non-zero, ip := operand |
| 15 | `LT`     | Pop b, pop a, push (a<b) |
| 16 | `LE`     | Pop b, pop a, push (a≤b) |
| 17 | `GT`     | Pop b, pop a, push (a>b) |
| 18 | `GE`     | Pop b, pop a, push (a≥b) |
| 19 | `EQ`     | Pop b, pop a, push (a==b) |
| 20 | `NEQ`    | Pop b, pop a, push (a!=b) |
| 21 | `LOCAL`  | Address-of a local slot |
| 22 | `LOAD`   | Load through pointer on top |
| 23 | `STORE`  | Store through pointer on top |

## Register-mode byte (`opr`)

| # | Name | Operand interpretation |
|---|---|---|
| 0 | `R_NONE` | No operand |
| 1 | `R_ABS` | Absolute code address |
| 2 | `R_IMM` | Immediate literal |
| 3 | `R_STACK` | Stack-relative offset from `bp` |
| 4 | `R_HEAP` | Heap-relative offset from `hp` base |

## Magic header rationale

Locating the magic with `find()` (rather than checking byte 0)
lets the bytecode file carry a shell `#!/usr/bin/env j0x` line for
self-execution; the loader skips past anything before `Jzero!!\0`.
