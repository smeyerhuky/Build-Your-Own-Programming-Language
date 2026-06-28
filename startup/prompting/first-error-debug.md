---
type: Setup Prompt Pattern
title: First-error debug — compiler build failures
description: Prompt templates for diagnosing and fixing errors during the first `make` run.
tags: [prompting, debug, build-error, first-build, troubleshoot]
timestamp: 2026-06-27T00:00:00Z
---

# First-error debug

Use these templates when `make` (or a manual build step) prints an
error you don't understand.

---

## Template: javac compilation error

```
I am building the Ch <N> J0 compiler. Running `make` (or `javac`) gave:
  <PASTE EXACT ERROR>

The file at fault is: <filename.java>
The lines around the error are:
  <PASTE RELEVANT CODE BLOCK>

Context:
- I generated Yylex.java with: jflex javalex.l
- I generated parser.java with: byacc -J j0gram.y
- JDK version: <javac -version>

What is likely wrong, and how do I fix it?
```

**Example:**
```
I am building the Ch 3 J0 compiler. Running `javac simple.java` gave:
  simple.java:12: error: cannot find symbol
    int tok = yylex.yylex();
                         ^
  symbol: method yylex()
  location: variable yylex of type Yylex

The file at fault is: simple.java
Context:
- I ran `jflex javalex.l` and Yylex.java was generated.
- JDK version: javac 17.0.9

What is likely wrong, and how do I fix it?
```

---

## Template: `cannot find symbol: parser.DO`

```
Running `javac Yylex.java` after `jflex javalex.l` gives:
  Yylex.java:NN: error: cannot find symbol
    return parser.DO;
                  ^

I have NOT run byacc yet (or I ran an old version of byacc).
What is the correct build order, and what command regenerates parser.java?
```

---

## Template: `make` fails silently / wrong output

```
I ran `make` in ch<N>/ and it completed with no errors, but running:
  java <MAIN_CLASS> hello.java
gave unexpected output:
  <PASTE OUTPUT>

Expected output:
  <PASTE EXPECTED>

Build environment:
- JDK: <version>
- jflex: <version>
- byacc: <version>

What should I check? Are there stale .class files?
```

---

## Tips for getting better answers

- Always paste the **exact** error message — do not paraphrase.
- Include the command you ran, not just "it didn't work".
- Paste the output of `make -n` (dry run) so the model sees your build order.
