---
type: File Inventory
title: "Ch 9 — Key files"
description: Source files in /ch9 with their purpose.
resource: /ch9/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch9`

| File | Purpose |
|---|---|
| `j0machine.java` / `j0machine.icn` | The VM: state + fetch + interp loop. Canonical. |
| `tac.java` / `tac.icn` | TAC instruction record (carried forward, defined here for the first time in code). |
| `address.java` / `address.icn` | Address descriptor: region + offset. |
| `tree.java` / `tree.icn` | AST extended with bytecode emission (`genfirst`, `genfollow`, `gentargets`, `gencode`). |
| `symtab.*`, `symtab_entry.*`, `typeinfo.*`, `arraytype.java`, `classtype.java`, `methodtype.java`, `parameter.java` | Carried forward from Ch 7. |
| `serial.*`, `token.*`, `j0.*`, `j0gram.y`, `javalex.l`, `Yylex.java`, `parser.java`, `parserVal.java`, `j0gram.icn`, `j0gram_tab.icn`, `javalex.icn`, `yyerror.*`, `makefile` | Lexer/parser/build carried forward. |
| `xy5.java`, `helloerror.java` | Helpers / test inputs. |
| `hello.java`, `arrtst.java`, `clstest.java`, `funtest.java` | End-to-end test programs. |
