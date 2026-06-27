---
type: Edge Case Catalog
title: "Ch 3 — Edge cases (negative examples)"
description: Common lexer bugs and a chain-of-thought walk to each fix.
resource: /ch3/javalex.l
tags: [edge-cases, cot, lexer, debugging]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 3 lexer

## 1. Keyword bound to IDENTIFIER

### Buggy input
`javalex.l` has the new keyword `do` declared **after** the
`{id}` rule:

```
{id}        { return j0.scan(parser.IDENTIFIER); }
"do"        { return j0.scan(parser.DO); }
```

### Symptom
`do { } while (...)` parses as `IDENTIFIER { ... }` and the parser
reports an "unexpected `{`".

### CoT trace
1. Parser says `IDENTIFIER`, not `DO` — the parser only sees what
   the lexer returns.
2. Look at the lexer: rule order matters for ties; for the input
   `do`, both `{id}` and `"do"` match the same length.
3. Flex breaks ties by **declaration order**, so `{id}` wins.
4. Move `"do"` rule **above** `{id}`.

### Fix
Reorder the rules:

```
"do"        { return j0.scan(parser.DO); }
{id}        { return j0.scan(parser.IDENTIFIER); }
```

### Lesson
Reserved words must precede the generic identifier rule. The
identifier rule is the catch-all for "word-shaped things"; anything
specific must come first.

---

## 2. Longest-match traps

### Buggy input
The lexer has `"<"` declared but not `"<="`.

### Symptom
`a <= b` lexes as `a < = b`, two tokens where one was expected.

### CoT trace
1. Each character read greedily extends the match while a rule still
   matches.
2. `"<"` matches `<`, but no rule matches `<=` further — so the
   longest match is just `<`.
3. The `=` is then matched by the assignment rule on its own.
4. Add an explicit `"<="` rule (anywhere in the rules section).

### Fix
```
"<="                   { return j0.scan(parser.LESSTHANOREQUAL);}
```

### Lesson
Longest-match works only over **declared** rules. A multi-char
operator must exist as its own rule; declaration order doesn't fix
the absence of the rule.

---

## 3. Unterminated comment

### Buggy input
A source file with `/* the rest of the file forgets the end marker`.

### Symptom
JFlex throws `java.lang.Error: Error: could not match input` at the
character after the `/*`.

### CoT trace
1. The rule `"/*"([^*]|"*"+[^/*])*"*"+"/"` requires `*/` to close.
2. With no `*/`, the rule can't complete — JFlex backtracks
   character-by-character and eventually hits the catch-all `.`
   rule, which calls `j0.lexErr`.
3. The error points at the character after `/*`, not the start of
   the comment — that's confusing.

### Fix (optional)
Add a friendlier diagnostic by detecting EOF inside an open comment
in `j0.comment()`.

### Lesson
A multi-line comment regex is fragile; consider switching to a
start-condition (`<COMMENT>` exclusive state) for clearer error
messages.

---

## 4. String literal with embedded quote

### Buggy input
```
print("he said \"hi\"");
```

### Symptom
The lexer matches `"he said \"` (everything up to the first
unescaped `"`), then leaves `hi\""` as bad input.

### CoT trace
1. The rule `\"[^\"]*\"` is by design: any non-quote character, no
   escapes.
2. J0 doesn't have string escapes, so this isn't a bug in J0 — but
   it *will* surprise anyone porting in a Java program.

### Fix (if you want escapes)
Change the rule to `\"([^"\\]|\\.)*\"` and add an unescaping pass in
`j0.scan`.

### Lesson
Lexers are the place to decide which character sequences are
"strings"; downstream phases assume the lexer's call is final.
