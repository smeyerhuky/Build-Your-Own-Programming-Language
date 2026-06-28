---
type: Phase Section
title: "Phase 3 — Deep dive: javalex.l and simple.java"
description: How javalex.l is structured, what simple.java does, and why the three-step build sequence is in that order.
tags: [phase, first-build, deep-dive, javalex, simple]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive — Phase 3: `javalex.l` and `simple.java`

## The three-step build and why order matters

```
step 1:  jflex javalex.l          → Yylex.java
step 2:  javac Yylex.java          ─┐
         javac simple.java          ├─ → *.class  (order within step 2
         javac token.java           ─┘             doesn't matter)
step 3:  java simple hello.java    → stdout
```

Step 1 must precede Step 2 because `javac` imports `Yylex`. Step 2
must precede Step 3 because `java` needs the `.class` files.

## Structure of `javalex.l`

```
%%
%int
id=([a-zA-Z_][a-zA-Z0-9_]*)
%%
"/*"([^*]|"*"+[^/*])*"*"+"/"   { j0.comment(); }
"//".*\r?\n                     { j0.comment(); }
[ \t\r\f]+                      { j0.whitespace(); }
\n                              { j0.newline(); }

"break"                  { return j0.scan(parser.BREAK); }
"class"                  { return j0.scan(parser.CLASS); }
...
"while"                  { return j0.scan(parser.WHILE); }

"<="                     { return j0.scan(parser.LESSTHANOREQUAL); }
"<"                      { return j0.scan(j0.ord("<")); }
...

{id}                     { return j0.scan(parser.IDENTIFIER); }
[0-9]+                   { return j0.scan(parser.INTLIT); }
\"[^\"]*\"               { return j0.scan(parser.STRINGLIT); }
.                        { j0.lexErr("Unrecognized char"); }
%%
```

Three `%%` markers separate: definitions / rules / code.

## `simple.java` driver loop

```java
yylexer = new Yylex(new FileReader(argv[0]));
int i;
while ((i = yylexer.yylex()) != Yylex.YYEOF) {
    System.out.println("token " + i + ": " + yytext());
}
```

`yylex()` returns the next token category (an `int`). `yytext()`
returns the matched lexeme. `YYEOF` is the sentinel that signals
end-of-file.

## Where line numbers come from

`j0.newline()` increments a static `line` counter. `j0.scan(cat)`
packages `line` into the returned token so error messages downstream
point at the right line. The `simple` driver doesn't use line numbers,
but they become important from Ch 4 onward.

## Cross-references

- [Token stream concept](../../concepts/token-stream.md)
- [JFlex tool card](../../tools/jflex.md)
- [Wiki deep dive: javalex.l](/wiki/chapters/ch03-lexical-analysis/deep-dive.md)
