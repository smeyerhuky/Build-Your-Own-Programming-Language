---
type: Chapter Section
title: "Ch 17 — Overview"
description: A capstone project shape and pointers to references.
tags: [overview, capstone]
timestamp: 2026-06-27T00:00:00Z
---

# Overview

The book's capstone suggests picking one missing feature (generics,
exceptions, a real GC, optimization pass, debugger), implementing it
end-to-end through every chapter's layer, and writing it up.

## Suggested capstones

- **Generics**: extend lexer (`<`/`>` as type brackets), grammar,
  symbol table (type parameters per scope), type checker (parameter
  substitution), codegen (monomorphization).
- **Exceptions**: extend grammar (`try`/`catch`/`throw`), AST,
  symtab, type system (exception types), codegen (exception
  tables), VM (unwind opcode).
- **JIT**: profile the VM, identify hot paths, emit native code for
  them.
