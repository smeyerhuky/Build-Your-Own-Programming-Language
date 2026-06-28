---
type: Phase Section
title: "Phase 4 — Concepts"
description: The cross-phase ideas the extend-J0 phase draws on.
tags: [phase, extend-j0, concepts, keyword, grammar]
timestamp: 2026-06-27T00:00:00Z
---

# Concepts — Phase 4: Extend J0

## The lexer–grammar contract

The lexer and grammar communicate via **token constants**. When the
lexer returns `parser.DO`, the grammar must have declared `%token DO`.
The constant's numeric value is automatically assigned by BYacc; it
does not matter to the programmer as long as both sides name the
same symbol.

**Pairing rule:** every new keyword or multi-character operator in
`javalex.l` needs a matching `%token` in `j0gram.y`.

Single-character operators (like `&`) are the exception: they use
`j0.ord("&")`, which returns the ASCII ordinal. The grammar refers to
them as the character literal `'&'` instead of a `%token` name.

## Rule ordering in the lexer

Lexers use **longest-match first, then declaration order** to break
ties. For a keyword:

- Input `do` → both `"do"` and `{id}` match 2 characters.
- Declaration order breaks the tie: the rule listed first wins.
- Therefore: **keywords must appear before `{id}`**.

For multi-character operators:

- Input `<=` → `"<="` matches 2 characters; `"<"` matches 1.
- Longest match wins automatically, regardless of declaration order.
- But if `"<="` is missing entirely, `"<"` grabs `<` alone (bug).

## J0 language concept

See [J0 language](../../concepts/j0-language.md) for the full
vocabulary — reserved words, operators, types, and statements.

## Related concept pages

- [J0 language](../../concepts/j0-language.md)
- [Token stream](../../concepts/token-stream.md)
- [Wiki: Lexical analysis](/wiki/concepts/lexical-analysis.md)
- [Wiki: Context-free grammar](/wiki/concepts/context-free-grammar.md)
