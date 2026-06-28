---
type: Phase Section
title: "Phase 4 — Checklist"
description: Step-by-step checklist for extending J0 with a new keyword and operator.
tags: [phase, extend-j0, checklist]
timestamp: 2026-06-27T00:00:00Z
---

# Checklist — Phase 4: Extend J0

---

## ☐ 1. Add the `do` keyword to the lexer

Open `ch3/javalex.l`. Find the block of keyword rules (lines with
`"break"`, `"bool"`, `"class"`, etc.). Insert **before** the `{id}`
identifier rule, in alphabetical position near `"double"`:

```
"do"                   { return j0.scan(parser.DO); }
```

**Important:** the rule must appear *before* the `{id}` rule. If it
appears after, the lexer returns `IDENTIFIER` instead of `DO` for the
text `do`.

---

## ☐ 2. Declare `DO` in the grammar

Open `ch4/j0gram.y`. Find the `%token` section near the top. Add:

```yacc
%token DO
```

Alphabetical position near `DOUBLE` is conventional but not required.

---

## ☐ 3. Add the `&` operator to the lexer

In `ch3/javalex.l`, in the single-character operator section (near
`"!"`, `"+"`, `"-"`, …):

```
"&"                    { return j0.scan(j0.ord("&")); }
```

Single-character operators use `j0.ord` to avoid allocating a named
token constant.

---

## ☐ 4. Rebuild the lexer

```bash
cd ch3
jflex javalex.l
javac Yylex.java simple.java token.java
```

No compile errors expected.

---

## ☐ 5. Test the keyword

```bash
printf 'public class T { public static void main(string[] a) { int x = 0; do {} } }\n' \
  > /tmp/keyword_test.java
java simple /tmp/keyword_test.java | grep -i 'do'
```

You should see a token for `do` (not `IDENTIFIER`). The exact token
number matches the `parser.DO` constant.

---

## ☐ 6. Test the operator

```bash
printf 'public class T { public static void main(string[] a) { int x = 1 & 2; } }\n' \
  > /tmp/op_test.java
java simple /tmp/op_test.java | grep '&'
```

You should see a token for `&`.

---

All checks pass → the language is extended. Proceed to
[Phase 5](../phase-05-full-pipeline/index.md) or experiment further
using the [cookbook](./exercises/cookbook.md).
