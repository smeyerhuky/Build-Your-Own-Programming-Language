---
type: Concept
title: Compiler pipeline
description: The end-to-end flow from source.j0 to either bytecode-on-VM or x86-64 native, mapped onto chapters.
tags: [pipeline, architecture, front-end, middle-end, back-end]
timestamp: 2026-06-27T00:00:00Z
---

# Compiler pipeline

## Front-end (Ch 3–7)

```
source.j0
  → lex     (ch3 javalex.l  → tokens)
  → parse   (ch4 j0gram.y   → parse tree)
  → AST     (ch5 tree.java  → AST)
  → scopes  (ch6 symtab.java → AST + symbol tables)
  → types   (ch7 typeinfo   → AST + symbol tables + types)
```

## Middle-end (Ch 8–9)

```
annotated AST
  → tac      (ch8 tac.java     → list of TAC instructions)
  → interp   (ch9 j0machine.java executes TAC directly OR
             ch9 bytecode generation walks AST → opcodes)
```

## Back-end variants (Ch 11–13)

```
annotated AST
  ├──▶ ch11/12  bytecode (.j0 file with "Jzero!!\0" magic, 8-byte ops)
  │             loaded by ch11/12 j0machine and run
  │
  └──▶ ch13     x86-64 assembly (x64.java emits, .L labels, AT&T syntax)
                assembled and linked by host toolchain
```

## Why two back-ends

The bytecode path lets you ship a self-contained `.j0` file and a
small interpreter; the native path produces faster code but ties you
to the host CPU and ABI. The course teaches both so the trade-off is
concrete.
