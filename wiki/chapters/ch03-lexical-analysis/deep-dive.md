---
type: Chapter Section
title: "Ch 3 — Deep dive: javalex.l"
description: Walks the three sections of /ch3/javalex.l and the rule ordering that makes it correct.
resource: /ch3/javalex.l
tags: [deep-dive, flex, javalex]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `javalex.l`

The full J0 lexer is one Flex file, ~58 lines. Three sections,
separated by `%%`.

## Section 1 — Definitions

```
%%
%int
id=([a-zA-Z_][a-zA-Z0-9_]*)
```

- `%int` tells JFlex the action functions return `int`.
- `id` names a regex macro reused later.

## Section 2 — Rules

Rules fire by **longest match first**, then **declaration order**.
Ordering is intentional:

1. Comments and whitespace fire first (highest priority that returns
   no token):
   ```
   "/*"([^*]|"*"+[^/*])*"*"+"/" { j0.comment(); }
   "//".*\r?\n                  { j0.comment(); }
   [ \t\r\f]+                   { j0.whitespace(); }
   \n                           { j0.newline(); }
   ```
2. Reserved words next, one rule each, returning a parser token:
   ```
   "break"                { return j0.scan(parser.BREAK); }
   "double"               { return j0.scan(parser.DOUBLE); }
   ...
   ```
3. Multi-character operators can be listed adjacent to their single-character prefixes; longest-match ensures `<=` is recognized even if `"<"` appears before it:
   ```
   "<="                   { return j0.scan(parser.LESSTHANOREQUAL);}
   ```
   (In this repo, `"<"` is listed before `"<="` in `javalex.l`.)
   characters.
4. Identifier rule `{id}` — must come **after** every keyword.
5. Numeric and string literal rules.
6. Catch-all `.` that calls `j0.lexErr` for unrecognized characters.

## Section 3 — Code

Empty in `/ch3/javalex.l`. Later chapters add helpers here.

## Where line numbers come from

`j0.newline()` increments a counter; `j0.scan(parser.X)` packages the
counter into the returned token so parser errors point at the right
line.

## Cross-references

- [Concept: Lexical analysis](/wiki/concepts/lexical-analysis.md)
- [Tool: Flex / Uflex](/wiki/tools/flex-uflex.md)
- [Language: J0](/wiki/languages/j0.md)
