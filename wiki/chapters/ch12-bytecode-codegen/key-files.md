---
type: File Inventory
title: "Ch 12 — Key files"
description: Source files in /ch12 with their purpose.
resource: /ch12/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch12`

| File | Purpose |
|---|---|
| `j0.java` / `j0.icn` | The compiler driver: end-to-end source → `.j0`. |
| `byc.java` / `byc.icn` | Bytecode writer (magic, entry, instructions, strings). |
| `j0machine.java` / `j0machine.icn` | Loader/runner (carried forward from Ch 11). |
| `j0x.java` / `j0x.icn` | Standalone executor. |
| `Op.java` / `Op.icn` | 23 opcodes + 5 register modes. |
| `address.java` / `address.icn`, `tac.java` / `tac.icn` | Address descriptor and IR instruction. |
| `tree.java` / `tree.icn` | AST with full bytecode emission. |
| `symtab.*`, `symtab_entry.*`, `typeinfo.*`, `arraytype.java`, `classtype.java`, `methodtype.java`, `parameter.java` | Carried forward. |
| `serial.*`, `token.*`, `j0gram.y`, `javalex.l`, `yyerror.*`, `makefile` | Carried forward. |
| `hello.java`, `hello.icn`, `hello.j0` | Test source + Unicon driver + reference bytecode. |
