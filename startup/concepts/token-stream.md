---
type: Setup Concept
title: Token stream
description: What a lexer produces — a stream of (category, lexeme) pairs that the parser consumes.
tags: [concept, tokens, lexer, scanner, token-stream]
timestamp: 2026-06-27T00:00:00Z
---

# Token stream

A **lexer** (scanner) reads a source file character by character and
emits a stream of **tokens**. A token is a `(category, lexeme)` pair:

- **Category** — an integer constant identifying the type of token
  (e.g., `parser.IF = 259`, `parser.IDENTIFIER = 258`).
- **Lexeme** — the original text that was matched (e.g., `"while"`,
  `"myVar"`, `42`).

The parser in Ch 4 only sees categories; it calls `yylex()` to get
the next category and uses `yytext()` to retrieve the lexeme when
needed.

## Example

Source text:
```java
int x = 42;
```

Token stream:
```
INT          int
IDENTIFIER   x
=            =
INTLIT       42
;            ;
```

## Where it comes from

`javalex.l` contains rules of the form `<regex> { <Java action> }`.
JFlex compiles these rules into `Yylex.java`. At runtime:

```java
int category = yylex.yylex();    // get next category
String lexeme = yylex.yytext();  // get matched text
```

## Why whitespace and comments disappear

Rules for whitespace and comments call `j0.whitespace()` or
`j0.comment()` — they do not return a token category. The lexer
silently skips them; the parser never sees them.

## See also

- [Wiki: Lexical analysis](/wiki/concepts/lexical-analysis.md)
- [Phase 3: First build](../phases/phase-03-first-build/index.md)
- [JFlex tool card](../tools/jflex.md)
