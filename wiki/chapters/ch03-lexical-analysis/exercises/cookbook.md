---
type: Cookbook
title: "Ch 3 — Cookbook (positive examples)"
description: Canonical lexer-extension patterns, shaped for few-shot prompting.
resource: /ch3/javalex.l
tags: [cookbook, few-shot, lexer]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 3 lexer

## 1. Add a single-character operator

### Prompt
"Add a bitwise-AND operator `&` to J0."

### Reference solution
In `/ch3/javalex.l`, in the operator section (next to `"!"`, `"+"`,
…) add:

```
"&"                    { return j0.scan(j0.ord("&")); }
```

The grammar later refers to it as the character literal `'&'`; no
new `%token` is needed because single-character operators are
represented by their ASCII ordinal.

### Why this is canonical
- Stays consistent with the existing one-char operator idiom.
- `j0.ord` keeps the token-id namespace small.

---

## 2. Add a multi-character operator

### Prompt
"Add `<<=` (shift-left-assign) to J0."

### Reference solution
1. In `/ch3/javalex.l`, **before** the single-character `"<"` rule:
   ```
   "<<="                  { return j0.scan(parser.SHLASSIGN); }
   ```
   (Longest-match still wins, but declaration order matters when two
   patterns match the same length.)
2. In the grammar's `%token` declarations, add `SHLASSIGN`.

### Why this is canonical
Multi-character operators follow the same Flex pattern; the grammar
must learn the new token name.

---

## 3. Add a keyword

### Prompt
"Add a `do` keyword for a future `do { } while` loop."

### Reference solution
1. In `/ch3/javalex.l`, **before** the `{id}` rule, alphabetically
   near `"double"`:
   ```
   "do"                   { return j0.scan(parser.DO); }
   ```
2. In `/ch4/j0gram.y`, declare it: `%token DO`.

### Why this is canonical
- Keywords must come **before** the generic identifier rule so they
  bind first.
- Pair every lexer change with a matching `%token` so Yacc knows the
  symbol.

---

## 4. Track column numbers

### Prompt
"Extend the scanner to track column numbers for error messages."

### Reference solution
In the `j0` helper class (called from every action), add a `col`
field. In `j0.scan`, increment `col` by `yyleng`. In `j0.newline`,
reset `col` to 1.

### Why this is canonical
The lexer is the only phase that sees raw characters; column
tracking belongs here, not in the parser or AST.
