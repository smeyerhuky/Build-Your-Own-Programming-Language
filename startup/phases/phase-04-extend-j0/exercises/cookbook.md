---
type: Cookbook
title: "Phase 4 — Cookbook (positive examples)"
description: Canonical J0-extension patterns for adding keywords and operators, shaped for few-shot prompting.
tags: [cookbook, extend-j0, positive-examples, keyword, operator]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Phase 4: Extend J0

## 1. Add a keyword

### Prompt
"Add a `do` keyword to J0 for a future `do-while` loop."

### Reference solution

**`ch3/javalex.l`** — before the `{id}` rule, alphabetically near `"double"`:
```
"do"                   { return j0.scan(parser.DO); }
```

**`ch4/j0gram.y`** — in the `%token` section:
```yacc
%token DO
```

Rebuild:
```bash
cd ch3 && jflex javalex.l && javac Yylex.java simple.java token.java
```

### Why this is canonical
- The lexer rule must precede `{id}` — keyword before identifier.
- Every new keyword needs a `%token` in the grammar so BYacc assigns
  a constant and `javalex.l` can reference `parser.DO`.

---

## 2. Add a single-character operator

### Prompt
"Add a bitwise-AND operator `&` to J0."

### Reference solution

**`ch3/javalex.l`** — in the single-character operator section:
```
"&"                    { return j0.scan(j0.ord("&")); }
```

No grammar change needed — single-char operators use their ASCII ordinal.

### Why this is canonical
`j0.ord` is the established idiom for single-character operators
(see existing `"!"`, `"+"`, etc. rules). No named `%token` keeps the
namespace clean.

---

## 3. Add a multi-character operator

### Prompt
"Add `&&` (logical AND) as a distinct token from `&`."

### Reference solution

**`ch3/javalex.l`** — **before** the single-character `"&"` rule:
```
"&&"                   { return j0.scan(parser.LOGICALAND); }
"&"                    { return j0.scan(j0.ord("&")); }
```

**`ch4/j0gram.y`**:
```yacc
%token LOGICALAND
```

### Why this is canonical
Multi-char operators need explicit rules and `%token` declarations.
Declaration order matters only for same-length matches; longest-match
automatically prefers `"&&"` over `"&"` for input `&&`.

---

## 4. Add a reserved word used as a type

### Prompt
"Add `long` as a primitive type keyword."

### Reference solution

**`ch3/javalex.l`** — before `{id}`, near `"int"`:
```
"long"                 { return j0.scan(parser.LONG); }
```

**`ch4/j0gram.y`**:
```yacc
%token LONG
```

Then add `LONG` to the `PrimitiveType` grammar rule (Ch 4) and a
corresponding entry in the type-checker (Ch 7).

### Why this is canonical
Types ripple further than operators — lexer + grammar + type-checker
must all be updated. Start with lexer and grammar (this phase); type-
checker changes belong in Ch 7 study.
