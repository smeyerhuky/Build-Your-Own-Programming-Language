---
type: Index
title: Chapters
description: Reading order and status for the 17 chapters of the course.
tags: [okf, chapters, index]
timestamp: 2026-06-27T00:00:00Z
---

# Chapters

Suggested reading order matches the numeric order. Each row links to a
chapter folder; descend into its `index.md` first.

| # | Chapter | Status | Folder |
|---|---|---|---|
| 1  | Why Build a Programming Language? | Book-only | [`ch01-fundamentals`](/wiki/chapters/ch01-fundamentals/index.md) |
| 2  | Programming-Language Design | Book-only | [`ch02-language-design`](/wiki/chapters/ch02-language-design/index.md) |
| 3  | Scanning Source Code | Full | [`ch03-lexical-analysis`](/wiki/chapters/ch03-lexical-analysis/index.md) |
| 4  | Parsing | Full | [`ch04-syntax-analysis`](/wiki/chapters/ch04-syntax-analysis/index.md) |
| 5  | Syntax Trees | Full | [`ch05-ast`](/wiki/chapters/ch05-ast/index.md) |
| 6  | Symbol Tables | Full | [`ch06-symbol-tables`](/wiki/chapters/ch06-symbol-tables/index.md) |
| 7  | Checking Base Types | Full | [`ch07-type-system`](/wiki/chapters/ch07-type-system/index.md) |
| 8  | Checking Types on Arrays, Method Calls and Structure Accesses | Full | [`ch08-intermediate-rep`](/wiki/chapters/ch08-intermediate-rep/index.md) |
| 9  | Intermediate Code Generation | Full | [`ch09-bytecode-vm`](/wiki/chapters/ch09-bytecode-vm/index.md) |
| 10 | Syntax-Coloring in an IDE | Book-only | [`ch10-bytecode-format`](/wiki/chapters/ch10-bytecode-format/index.md) |
| 11 | Bytecode Interpreters | Full | [`ch11-bytecode-serialization`](/wiki/chapters/ch11-bytecode-serialization/index.md) |
| 12 | Generating Bytecode | Full | [`ch12-bytecode-codegen`](/wiki/chapters/ch12-bytecode-codegen/index.md) |
| 13 | Native Code Generation | Full | [`ch13-native-codegen-x86`](/wiki/chapters/ch13-native-codegen-x86/index.md) |
| 14 | Preprocessors and Transpilers | Book-only | [`ch14-optimization`](/wiki/chapters/ch14-optimization/index.md) |
| 15 | Garbage Collection | Book-only | [`ch15-parsing-optimization`](/wiki/chapters/ch15-parsing-optimization/index.md) |
| 16 | Final Thoughts | Book-only | [`ch16-language-design-principles`](/wiki/chapters/ch16-language-design-principles/index.md) |
| 17 | Appendix / Capstone | Book-only | [`ch17-capstone`](/wiki/chapters/ch17-capstone/index.md) |

Folder slugs are kept stable across the wiki; chapter titles above are
the broad themes the chapters cover in the book — exact titles may
vary between book editions.

## Where the code starts and ends

The repo ships `/ch3` through `/ch13` with `/ch10`, `/ch14`–`/ch17`
omitted. The wiki keeps a folder for every chapter so cross-references
and book study still resolve, even when no source is present.
