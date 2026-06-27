---
type: File Inventory
title: "Ch 5 — Key files"
description: Source files in /ch5 with their purpose.
resource: /ch5/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch5`

| File | Purpose |
|---|---|
| `tree.java` / `tree.icn` | The AST node class (n-ary, with print + print_graph). Canonical. |
| `serial.java` / `serial.icn` | Unique id generator for DOT output. |
| `token.java` / `token.icn` | Token class embedded in leaf AST nodes. |
| `prodrule.java` | Production-rule descriptor used for nice tree labels. |
| `parserVal.java` | Yacc's value stack now carrying `tree` references. |
| `parser.java` | Yacc-generated parser, with semantic actions building the AST. |
| `j0gram.y` | Grammar with semantic actions added. |
| `javalex.l`, `javalex.icn` | Lexer carried forward. |
| `j0.java` / `j0.icn`, `j0gram.icn`, `j0gram_tab.icn`, `Yylex.java` | Driver + Unicon outputs + lexer artifacts. |
| `xy5.java`, `cereal.icn` | Helpers used by examples. |
| `hello.java`, `helloerror.java` | Test inputs. |
| `yyerror.java` / `yyerror.icn`, `makefile` | Error reporter + build glue. |
