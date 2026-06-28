---
type: Setup Concept
title: J0 language
description: What J0 is — a deliberately small Java subset used as the source language throughout the compiler course.
tags: [concept, j0, source-language, java-subset]
timestamp: 2026-06-27T00:00:00Z
---

# J0

J0 is a **deliberately small Java subset**. It is small enough that a
complete compiler can be built in a single book, yet large enough to
illustrate every important compiler phase.

See the full spec at [wiki: J0 language](/wiki/languages/j0.md).

## What J0 has

- One top-level class per file.
- `public static void main(string[] argv)` entry point.
- Primitive types: `int`, `double`, `bool`, `string`, `void`.
- Statements: `if/else`, `while`, `for`, `break`, `return`.
- Expressions: arithmetic, comparison, logical, assignment.
- A synthetic `System.out.println()` (no real standard library).

## What J0 intentionally omits

- Generics, interfaces, abstract classes.
- Exceptions.
- Implicit numeric widening on assignment.
- A real standard library beyond `System.out.println`.

## Why this matters for onboarding

Every file you lex, parse, and compile is a J0 file. `hello.java`
is the canonical test case — 5 lines, exercises keywords, a string
literal, and method-call syntax.

```java
public class hello {
  public static void main(string[] argv) {
    System.out.println("hello, world");
  }
}
```

## Hello, world token stream (Ch 3 output)

```
token 258: public
token 258: class
token 258: hello
token 273: {
token 258: public
token 258: static
token 258: void
token 258: main
...
token 295: "hello, world"
...
```
