---
type: File Inventory
title: "Ch 8 — Key files"
description: Source files in /ch8 with their purpose.
resource: /ch8/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch8`

| File | Purpose |
|---|---|
| `tac.java` / `tac.icn` (in `/ch9` onward) | TAC instruction record. Note: `tac.java` first ships in `/ch9` and is **used** at Ch 8 level conceptually; `/ch8` introduces the concept and the surrounding test programs. |
| `tree.java` / `tree.icn` | AST extended with `icode` and `gencode()`. |
| `symtab.*`, `typeinfo.*`, `arraytype.java`, `classtype.java`, `methodtype.java`, `parameter.java` | Carried forward from Ch 7. |
| `serial.*`, `token.*`, `j0.*`, `j0gram.y`, `javalex.l`, `yyerror.*`, `xy5.java`, `makefile` | Build/glue/lexer carried forward. |
| `hello.java` | Hello-world test input. |
| `arrtst.java` | Array-operations test. |
| `clstest.java` | Class-declaration test. |
| `funtest.java` | Function/method test. |
