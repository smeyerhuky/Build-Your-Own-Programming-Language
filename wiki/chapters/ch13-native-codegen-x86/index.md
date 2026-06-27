---
type: Chapter
title: "Ch 13 — Native Code Generation (x86-64)"
description: AST → AT&T-syntax x86-64 assembly with a register allocator and SysV calling convention.
resource: /ch13/
tags: [chapter, native, x86-64, register-allocation, x64]
timestamp: 2026-06-27T00:00:00Z
---

# Ch 13 — Native Code Generation (x86-64)

The second back-end. The same typed AST now lowers to x86-64 assembly
that can be assembled and linked into a native binary.

## Children

- [Overview](/wiki/chapters/ch13-native-codegen-x86/overview.md)
- [Concepts](/wiki/chapters/ch13-native-codegen-x86/concepts.md)
- [Key files](/wiki/chapters/ch13-native-codegen-x86/key-files.md)
- [Deep dive: `x64.java` & `x64loc.java`](/wiki/chapters/ch13-native-codegen-x86/deep-dive.md)
- [Exercises](/wiki/chapters/ch13-native-codegen-x86/exercises/index.md)
