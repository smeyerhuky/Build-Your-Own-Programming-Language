---
type: File Inventory
title: "Ch 6 — Key files"
description: Source files in /ch6 with their purpose.
resource: /ch6/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch6`

| File | Purpose |
|---|---|
| `symtab.java` / `symtab.icn` | The symbol-table itself: HashMap + parent + scope name. Canonical. |
| `symtab_entry.java` / `symtab_entry.icn` | One row per declared name; carries optional sub-scope. |
| `tree.java` / `tree.icn` | AST extended with `st` field plus `mkSymTables`, `populateSymTables`, `checkSymTables`. |
| `serial.java` / `serial.icn` | Unique id generator carried forward. |
| `token.java` / `token.icn` | Token class. |
| `j0.java` / `j0.icn` | Driver; installs `System` built-in. |
| `j0gram.y`, `javalex.l` | Grammar and lexer carried forward. |
| `hello.java`, `helloerror.java`, `xy5.java` | Test inputs. |
| `yyerror.java` / `yyerror.icn`, `makefile` | Error reporter + build glue. |
