---
type: File Inventory
title: "Ch 4 — Key files"
description: Source files in /ch4 with their purpose.
resource: /ch4/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch4`

| File | Purpose |
|---|---|
| `j0gram.y` | Full J0 grammar, no semantic actions yet. Canonical artifact. |
| `nameseq.y`, `ns.y` | Warm-up grammars (sequences of names). |
| `Parser.java`, `parserTokens.java`, `parserVal.java` | Yacc output for J0. |
| `Yylex.java` | JFlex output regenerated from `javalex.l`. |
| `j0.java` / `j0.icn` | Driver that calls scanner + parser and reports success/failure. |
| `j0go.java` / `j0go.icn` | Alternative driver used in examples. |
| `lexer.java` | Glue between JFlex and Yacc's expected lexer interface. |
| `trivial.java` / `trivial.icn` | Smallest parser example. |
| `yyerror.java` / `yyerror.icn` | Error-reporting helper. |
| `javalex.l`, `javalex.icn`, `nnws.l` | Lexer specs carried forward from Ch 3. |
| `j0gram.icn`, `j0gram_tab.icn` | Iyacc output (Unicon). |
| `hello.java`, `helloerror.java`, `dorrie3.in`, `jzero.java`, `ns.icn`, `ns_tab.icn` | Test inputs and Unicon equivalents. |
| `token.java` | Token record. |
