---
type: RFC
title: "RFC-002 — J0 Bytecode Format"
description: Specification of the J0 bytecode file format including file layout, opcode table, and encoding rules.
tags: [rfc, bytecode, j0, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
rfc_number: "002"
status: Accepted
author: Clinton L. Jeffery
---

# RFC-002 — J0 Bytecode Format

| Field | Value |
|---|---|
| RFC Number | 002 |
| Status | Accepted |
| Author | Clinton L. Jeffery |
| Created | 2026-06-28 |
| Last updated | 2026-06-28 |

---

## Summary

This RFC specifies the binary format of `.j0` bytecode files produced by the J0 compiler and consumed by the J0 bytecode interpreter. It covers the file header, constant pool, method table, and 23-opcode instruction set.

---

## 1. File structure

A `.j0` file has the following top-level sections in order:

```
+------------------+
|   File header    |  (magic, version, counts)
+------------------+
|  Constant pool   |  (string, integer, class, method constants)
+------------------+
|   Class table    |  (class name, field count, method refs)
+------------------+
|   Method table   |  (method name, descriptor, bytecode)
+------------------+
```

All multi-byte integers are stored **big-endian** (network byte order).

---

## 2. File header

| Offset | Size | Field | Value |
|---|---|---|---|
| 0 | 4 bytes | Magic number | `0x4A 0x30 0x42 0x43` ("J0BC") |
| 4 | 2 bytes | Major version | `1` |
| 6 | 2 bytes | Minor version | `0` |
| 8 | 2 bytes | Constant pool count | number of constant pool entries |
| 10 | 2 bytes | Class count | number of class table entries |
| 12 | 2 bytes | Method count | number of method table entries |

Total header size: 14 bytes.

---

## 3. Constant pool

Each constant pool entry begins with a 1-byte tag:

| Tag | Name | Encoding |
|---|---|---|
| `0x01` | `CONST_INT` | tag (1) + value (4, signed big-endian int32) |
| `0x02` | `CONST_STRING` | tag (1) + length (2) + UTF-8 bytes (length) |
| `0x03` | `CONST_CLASS` | tag (1) + name_index (2, index into constant pool) |
| `0x04` | `CONST_METHOD_REF` | tag (1) + class_index (2) + name_index (2) + descriptor_index (2) |
| `0x05` | `CONST_FIELD_REF` | tag (1) + class_index (2) + name_index (2) |

Constant pool indices are 1-based (index 0 is reserved and always null).

---

## 4. Class table

Each class entry:

| Size | Field |
|---|---|
| 2 bytes | `name_index` — index of class name in constant pool |
| 2 bytes | `field_count` — number of instance fields |
| 2 bytes | `method_count` — number of methods defined in this class |
| method_count × 2 bytes | `method_indices` — indices into the method table |

---

## 5. Method table

Each method entry:

| Size | Field |
|---|---|
| 2 bytes | `name_index` — method name in constant pool |
| 2 bytes | `descriptor_index` — descriptor string (e.g., `(II)V`) in constant pool |
| 2 bytes | `max_stack` — maximum operand stack depth |
| 2 bytes | `max_locals` — number of local variable slots |
| 4 bytes | `code_length` — number of bytes of bytecode |
| code_length bytes | `code` — bytecode instructions |

---

## 6. Instruction set (23 opcodes)

### Stack manipulation

| Opcode | Hex | Operands | Description |
|---|---|---|---|
| `ICONST` | `0x01` | value (4) | Push integer constant onto stack |
| `SCONST` | `0x02` | pool_index (2) | Push string constant (from constant pool) |
| `ACONST_NULL` | `0x03` | — | Push null reference |
| `POP` | `0x04` | — | Pop and discard top of stack |
| `DUP` | `0x05` | — | Duplicate top of stack |

### Local variable access

| Opcode | Hex | Operands | Description |
|---|---|---|---|
| `ILOAD` | `0x10` | index (2) | Push integer local variable at index |
| `ALOAD` | `0x11` | index (2) | Push reference local variable at index |
| `ISTORE` | `0x12` | index (2) | Pop and store to integer local variable |
| `ASTORE` | `0x13` | index (2) | Pop and store to reference local variable |

### Arithmetic and comparison

| Opcode | Hex | Operands | Description |
|---|---|---|---|
| `IADD` | `0x20` | — | Pop two ints, push their sum |
| `ISUB` | `0x21` | — | Pop two ints, push their difference (lower − upper) |
| `IMUL` | `0x22` | — | Pop two ints, push their product |
| `IDIV` | `0x23` | — | Pop two ints, push their quotient |
| `IREM` | `0x24` | — | Pop two ints, push their remainder |
| `ICMP` | `0x25` | — | Pop two ints; push -1, 0, or 1 |

### Control flow

| Opcode | Hex | Operands | Description |
|---|---|---|---|
| `GOTO` | `0x30` | offset (2, signed) | Unconditional branch by offset from current PC |
| `IFEQ` | `0x31` | offset (2, signed) | Pop int; branch if == 0 |
| `IFNE` | `0x32` | offset (2, signed) | Pop int; branch if != 0 |
| `IFLT` | `0x33` | offset (2, signed) | Pop int; branch if < 0 |
| `IFGT` | `0x34` | offset (2, signed) | Pop int; branch if > 0 |

### Object and method operations

| Opcode | Hex | Operands | Description |
|---|---|---|---|
| `INVOKEVIRTUAL` | `0x40` | method_ref_index (2) | Invoke instance method; pops args + receiver |
| `INVOKESTATIC` | `0x41` | method_ref_index (2) | Invoke static method; pops args |
| `RETURN` | `0x50` | — | Return void from current method |
| `IRETURN` | `0x51` | — | Pop int and return it from current method |
| `ARETURN` | `0x52` | — | Pop reference and return it |
| `NEW` | `0x60` | class_index (2) | Allocate new object; push reference |
| `GETFIELD` | `0x61` | field_ref_index (2) | Pop object ref; push field value |
| `PUTFIELD` | `0x62` | field_ref_index (2) | Pop value and object ref; store field |
| `NEWARRAY` | `0x63` | type_tag (1) | Pop length; allocate array; push reference |
| `AALOAD` | `0x64` | — | Pop index and array ref; push element |
| `AASTORE` | `0x65` | — | Pop value, index, array ref; store element |

Total distinct opcodes: 23 core + 7 extended array/field ops = **30 opcodes in full implementation**. The core 23 opcodes (sufficient for Chapters 8–9) are the ones originally specified in the book.

---

## 7. Descriptor format

Method descriptors use a compact notation:

- `(` *param-types* `)` *return-type*
- `I` = `int`, `Z` = `boolean`, `V` = `void`, `L`*ClassName*`;` = reference type, `[`*type* = array

Example: a method `int add(int a, int b)` has descriptor `(II)I`.

---

## Drawbacks

The 23-opcode set is intentionally minimal. It is not JVM-compatible — programs compiled to `.j0` bytecode cannot run in a JVM. This is intentional: the format is a teaching tool, not a production target.

## Alternatives considered

- **Use JVM bytecode directly:** Too complex for learning; requires a full JVM toolchain.
- **Use a text-based IR:** Easier to debug but harder to teach serialization/deserialization concepts.

## Unresolved questions

- Should `INVOKEVIRTUAL` support dynamic dispatch (virtual method tables)? Currently it uses static dispatch. Deferred to RFC-005.
