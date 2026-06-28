---
type: Edge Case Catalog
title: "Phase 4 — Edge cases (negative examples)"
description: Common J0-extension bugs with CoT traces and fixes.
tags: [edge-cases, extend-j0, negative-examples, debugging, keyword, grammar]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Phase 4: Extend J0

## 1. Keyword tokenized as IDENTIFIER

### Buggy input
Added `"do"` rule to `javalex.l` **after** the `{id}` rule:
```
{id}    { return j0.scan(parser.IDENTIFIER); }
"do"    { return j0.scan(parser.DO); }    ← WRONG position
```

### Symptom
Input `do { }` produces token `IDENTIFIER` for `do`.

### CoT trace
1. Both `{id}` and `"do"` match the 2-character input `do`.
2. Longest-match tie → JFlex breaks it by declaration order.
3. `{id}` was declared first → `{id}` wins.
4. Move `"do"` rule above `{id}`.

### Fix
```
"do"    { return j0.scan(parser.DO); }    ← before {id}
{id}    { return j0.scan(parser.IDENTIFIER); }
```

### Lesson
Reserved words must precede the generic identifier rule. JFlex's
longest-match rule makes keywords lose to `{id}` when both match
the same input length, unless the keyword comes first.

---

## 2. `cannot find symbol: variable DO in class parser` (compile error)

### Symptom
```
$ jflex javalex.l
Yylex.java:42: error: cannot find symbol
   return j0.scan(parser.DO);
                        ^
symbol: variable DO in class parser
```

### CoT trace
1. `jflex javalex.l` generates `Yylex.java` which references `parser.DO`.
2. `parser.DO` doesn't exist yet — `%token DO` was not added to
   `j0gram.y`, so `parser.java` was never regenerated with `DO`.
3. Even if `parser.java` was regenerated, `javac` compiled the old
   `parser.class` — the stale `.class` doesn't have `DO`.

### Fix
```bash
cd ch4 && byacc -J j0gram.y && cd ../ch3
jflex javalex.l
javac Yylex.java simple.java token.java
```
Always regenerate `parser.java` from `j0gram.y` before regenerating
`Yylex.java`.

### Lesson
The correct update order is: grammar → BYacc → JFlex → javac. Skipping
any step leaves stale generated files with inconsistent constants.

---

## 3. `&&` tokenized as two `&` tokens

### Buggy input
Added `"&"` but not `"&&"` to `javalex.l`. Input `a && b`.

### Symptom
Token stream shows: `IDENTIFIER`, `&`, `&`, `IDENTIFIER` (two single-amp tokens).

### CoT trace
1. Longest-match only operates over **declared rules**.
2. `"&&"` is not declared → JFlex's longest match is the single `"&"`.
3. Two `&` tokens are emitted for `&&`.

### Fix
Add the `"&&"` rule **before** the `"&"` rule:
```
"&&"    { return j0.scan(parser.LOGICALAND); }
"&"     { return j0.scan(j0.ord("&")); }
```

### Lesson
Multi-character tokens must be explicitly declared. Longest-match is
not magic — it only extends a match over declared rules, not over
arbitrary prefixes.

---

## 4. BYacc shift/reduce conflict on new token

### Symptom
```
$ byacc -J j0gram.y
1 shift/reduce conflict.
```

### CoT trace
1. Adding `%token DO` without adding a grammar rule that uses it
   causes no conflict on its own.
2. A conflict arises when the rule that uses `DO` is ambiguous — for
   example, a `do-while` with a dangling-else–like ambiguity.
3. BYacc resolves shift/reduce conflicts in favor of shift (the
   standard behavior), which is usually correct.

### Fix
Inspect the conflict report: `byacc -J -v j0gram.y` produces
`y.output` with the conflict details. Add explicit precedence or
restructure the rule to remove the ambiguity.

### Lesson
A shift/reduce warning on a new rule is expected in some cases
(dangling else is the classic example). Read `y.output` before
deciding to ignore or fix it.
