---
type: File Inventory
title: "Ch 13 — Key files"
description: Source files in /ch13 with their purpose.
resource: /ch13/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch13`

| File | Purpose |
|---|---|
| `j0.java` / `j0.icn` | Compiler driver with native codegen. |
| `x64.java` / `x64.icn` | One assembly instruction (`op opnd1, opnd2`). |
| `x64loc.java` / `x64loc.icn` | One operand: 6 modes (reg, int, disp, label-disp, immediate, label). |
| `RegUse.java` / `reguse.icn` | Register-usage bitset + allocator helpers. |
| `byc.java` / `byc.icn` | Assembly output writer. |
| `Op.java` / `Op.icn` | Opcode constants (carried forward; bytecode path coexists). |
| `address.*`, `tac.*` | IR descriptors. |
| `tree.java` / `tree.icn` | AST with x86-64 emission. |
| `symtab.*`, `symtab_entry.*`, `typeinfo.*`, `arraytype.java`, `classtype.java`, `methodtype.java`, `parameter.java` | Carried forward. |
| `j0machine.java` / `j0machine.icn`, `j0x.java` / `j0x.icn` | Bytecode side still present so both paths coexist. |
| `serial.*`, `token.*`, `j0gram.y`, `javalex.l`, `yyerror.*`, `makefile` | Build/glue. |
| `hello.java` | Test source. |
