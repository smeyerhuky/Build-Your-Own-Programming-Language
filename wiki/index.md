---
type: Course Overview
title: "Build Your Own Programming Language — OKF Wiki"
description: LLM-navigable knowledge bundle for Clinton L. Jeffery's compiler-construction course. Walks the 17 book chapters, the J0 source language, the J0 bytecode VM, and the x86-64 backend, with per-chapter cookbook (positive) and edge-case (negative) prompt material.
resource: /README.md
tags: [okf, compilers, programming-languages, j0, unicon, course, llm-wiki]
timestamp: 2026-06-27T00:00:00Z
---

# Build Your Own Programming Language — OKF Wiki

This bundle is an **Open Knowledge Format** wrapper around the Packt repo
*Build Your Own Programming Language* by Clinton L. Jeffery. It is
designed so an LLM (or a human) can walk the folder, never load more
than they need, and assemble accurate few-shot / chain-of-thought
prompts for compiler-construction tasks.

## How to read this bundle (progressive disclosure)

1. Start here.
2. Open [`/wiki/chapters/index.md`](/wiki/chapters/index.md) to see the
   17-chapter table. Each row links to one chapter folder.
3. Inside a full chapter folder, the entry point is `index.md`. From
   there you can descend into `overview.md`, `concepts.md`,
   `key-files.md`, `deep-dive.md`, or `exercises/`.
4. Cross-chapter ideas (lexing, AST, type systems, VMs) live under
   [`/wiki/concepts/`](/wiki/concepts/index.md). Chapter pages link
   there so you can pull a concept page without descending the chapter
   tree.
5. Languages and tools the course depends on live under
   [`/wiki/languages/`](/wiki/languages/index.md) and
   [`/wiki/tools/`](/wiki/tools/index.md).
6. The LLM-prompting harness — few-shot and chain-of-thought templates,
   and an aggregated catalog of negative examples across chapters —
   lives under [`/wiki/prompting/`](/wiki/prompting/index.md).

## Producer-defined OKF types used in this bundle

OKF requires only a `type` field; producers define the vocabulary. This
bundle uses:

| Type | Meaning |
|---|---|
| `Course Overview` | The bundle root. |
| `Index` | A directory `index.md` whose purpose is navigation. |
| `Chapter` | A chapter `index.md`. |
| `Chapter Section` | `overview.md`, `concepts.md`, `key-files.md`, `deep-dive.md`. |
| `Concept` | A cross-chapter concept page in `/wiki/concepts/`. |
| `Language Spec` | A page in `/wiki/languages/`. |
| `Tool` | A page in `/wiki/tools/`. |
| `Prompt Pattern` | Reusable few-shot / CoT templates. |
| `Cookbook` | Per-chapter positive examples for few-shot prompting. |
| `Edge Case Catalog` | Per-chapter negative examples for CoT debugging. |
| `File Inventory` | A `key-files.md` listing of source files in a chapter. |
| `Log` | The bundle's `log.md`. |

Consumers tolerate unknown types per the OKF spec.

## Repo status

| Range | Status | Notes |
|---|---|---|
| Ch 1–2 | Book-only | Background and language-design framing. |
| Ch 3–9 | Full | Code shipped in `/ch3` … `/ch9`. |
| Ch 10 | Book-only | Theory chapter; runtime/format implementation appears in `/ch11`. |
| Ch 11–13 | Full | Code shipped in `/ch11`, `/ch12`, `/ch13`. |
| Ch 14–17 | Book-only | Optimization, parsing optimization, language-design principles, capstone. |

## Pipeline at a glance

```
source.j0
   │  ch3  Lexer (Flex / Uflex .l → tokens)
   ▼
 tokens
   │  ch4  Parser (Yacc / Iyacc .y → parse tree)
   ▼
 parse tree
   │  ch5  AST construction (tree.java / tree.icn)
   ▼
 AST
   │  ch6  Symbol tables (symtab.java)
   │  ch7  Type checking (typeinfo, classtype, methodtype)
   ▼
 annotated AST
   │  ch8  Three-address code (tac.java)
   ▼
 TAC
   ├──▶ ch9  Stack VM interpretation (j0machine.java)
   ├──▶ ch11 Bytecode serialization (.j0 file, magic "Jzero!!\0")
   ├──▶ ch12 Direct AST → bytecode (byc.java)
   └──▶ ch13 AST → x86-64 (x64.java, x64loc.java, RegUse.java)
```

## License

Source code is licensed by Packt (see `/LICENSE`). This wiki is a
derived knowledge bundle and references the source by path; it does
not fork or vendor any source content.
