---
type: Chapter Section
title: First Build
description: Step-by-step walkthrough of building and running the Ch 3 J0 lexer — the "hello, world" of this compiler course.
tags: [startup, first-build, ch3, lexer, jflex]
timestamp: 2026-06-27T00:00:00Z
---

# First build — the Ch 3 lexer

Complete [Environment setup](./environment-setup.md) before this page.

This walkthrough compiles the Ch 3 lexer and runs it against
`hello.java`, the simplest J0 program. If you get clean token output,
your toolchain is ready for every later chapter.

---

## What you are building

```
hello.java  (source)
     │
     ▼  jflex javalex.l → Yylex.java
  Yylex.java + simple.java
     │
     ▼  javac → simple.class + Yylex.class
     │
     ▼  java simple hello.java → token stream
```

The `simple` driver calls `yylex()` in a loop and prints every token
number and its lexeme.

---

## Step 1 — Enter the chapter directory

```bash
cd ch3
```

---

## Step 2 — Generate the lexer

```bash
jflex javalex.l
```

Expected output (no errors):

```
Reading "javalex.l"
Constructing NFA : 107 states in NFA
Converting NFA to DFA :
...
Writing code to "Yylex.java"
```

You should now have `Yylex.java` in the current directory.

---

## Step 3 — Compile

```bash
javac Yylex.java simple.java token.java
```

This produces `Yylex.class`, `simple.class`, and `token.class`.

> **Tip:** if the chapter ships a `makefile`, `make` runs all three
> steps for you: `make simple`.

---

## Step 4 — Run against `hello.java`

```bash
java simple hello.java
```

Expected output (token numbers and lexemes):

```
token 258: public
token 258: class
token 258: hello
token 273: {
token 258: public
token 258: static
token 258: void
token 258: main
token 273: (
token 258: string
token 290: [
token 291: ]
token 258: argv
token 274: )
token 273: {
token 258: System
token 278: .
token 258: out
token 278: .
token 258: println
token 273: (
token 295: "hello, world"
token 274: )
token 279: ;
token 274: }
token 274: }
```

Token numbers correspond to constants defined in `token.java`; exact
values may vary slightly between editions but every keyword, operator,
and literal should appear.

---

## Step 5 — Run against your own file

Create a minimal J0 class:

```java
public class greet {
  public static void main(string[] argv) {
    int x = 42;
    System.out.println("done");
  }
}
```

Save it as `greet.java` and lex it:

```bash
java simple greet.java
```

You should see tokens for `int`, `x`, `=`, `42`, `;`, and so on.

---

## Unicon variant

If you prefer the Unicon path:

```bash
uflex javalex.l          # produces javalex.icn
unicon simple.icn hello.java
```

Output should be identical in structure.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `cannot find symbol: class Yylex` | Run `jflex javalex.l` first; `Yylex.java` must exist before `javac`. |
| `error: cannot find symbol: class token` | Compile all three files together: `javac Yylex.java simple.java token.java`. |
| No output / immediate exit | Pass the filename as an argument: `java simple hello.java`. |
| `illegal character` in lexer output | The input file may have Windows line endings; try `dos2unix hello.java`. |

---

## Next steps

Your lexer works. Move on to Ch 4 to add a parser, or read the
[Roadmap](./roadmap.md) to plan your full learning path.
