---
type: File Inventory
title: "Ch 3 — Key files"
description: Source files in /ch3 with their purpose.
resource: /ch3/
tags: [files, inventory]
timestamp: 2026-06-27T00:00:00Z
---

# Key files in `/ch3`

| File | Purpose |
|---|---|
| `javalex.l` | Flex spec for the full J0 lexer. The canonical artifact. |
| `nnws.l`, `nnws-tok.l` | Toy "non-whitespace" lexers used as warm-ups. |
| `simple.java` / `simple.icn` | Smallest possible scanner driver. |
| `simple2.java` / `simple2.icn` | Slight elaboration of `simple` showing token printing. |
| `j0.java` / `j0.icn` | The main `j0` driver — opens a file, runs the scanner in a loop, prints results. |
| `j0go.java` / `j0go.icn` | Alternative `j0` driver used by examples. |
| `parser.java` | Stub parser providing the token-constant namespace (`parser.IF`, `parser.LOGICALAND`, …) consumed by `javalex.l`. |
| `token.java` | Token record. |
| `hello.java` | J0 source: classic "hello, world". Lexer input. |
| `dorrie.in`, `dorrie2.in` | Plain-text fixtures for the warm-up scanners. |
