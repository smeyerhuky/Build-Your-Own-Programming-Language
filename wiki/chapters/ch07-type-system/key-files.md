---
type: File Inventory
title: "Ch 7 — Key files"
description: Source files in /ch7 with their purpose.
resource: /ch7/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch7`

| File | Purpose |
|---|---|
| `typeinfo.java` / `typeinfo.icn` | Primitive type descriptor (kind + optional sub-types). |
| `arraytype.java` | Wraps a base type plus dimension count. |
| `classtype.java` | Class name + symbol table of fields/methods. |
| `methodtype.java` | Return type + `Parameter[]` signature. |
| `parameter.java` | One formal parameter (name + type). |
| `tree.java` / `tree.icn` | AST extended with `typ`, `isConst`, plus calctype/checktype/assigntype. |
| `symtab.java`, `symtab_entry.java` | Symbol tables carried forward (now carry type info per entry). |
| `j0.java` / `j0.icn`, `j0gram.y`, `javalex.l` | Driver/grammar/lexer carried forward. |
| Other (`Yylex.java`, `parserVal.java`, `serial.java`, `token.java`, `xy5.java`, `yyerror.java`, `makefile`, …) | Build and helper files. |
| `hello.java`, `helloerror.java` | Test inputs. |
