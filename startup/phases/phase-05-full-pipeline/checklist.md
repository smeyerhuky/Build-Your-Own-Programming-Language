---
type: Phase Section
title: "Phase 5 — Checklist"
description: Step-by-step checklist for running the complete J0 compiler pipeline through the VM interpreter and optionally the x86-64 native backend.
tags: [phase, full-pipeline, checklist]
timestamp: 2026-06-27T00:00:00Z
---

# Checklist — Phase 5: Full pipeline

This phase spans Chapters 4–9 (and optionally 13). Work through
each chapter using its `makefile` and `hello.java` as the test input.

---

## ☐ 1. Build and test the Ch 4 parser

```bash
cd ch4
make
java ch4.j0 ../ch3/hello.java    # parses without syntax errors
```

A successful parse produces no output (or a parse-success message
depending on the edition). Any "syntax error" output means `j0gram.y`
did not compile correctly.

---

## ☐ 2. Build and test Ch 5 AST

```bash
cd ch5
make
java j0 ../ch3/hello.java    # prints or draws the AST
```

---

## ☐ 3. Build and test Ch 6 symbol tables

```bash
cd ch6
make
java j0 ../ch3/hello.java    # no "undeclared variable" errors
```

---

## ☐ 4. Build and test Ch 7 type checker

```bash
cd ch7
make
java j0 ../ch3/hello.java    # no type errors
```

---

## ☐ 5. Build and test Ch 8 TAC generation

```bash
cd ch8
make
java j0 ../ch3/hello.java    # prints TAC instructions
```

---

## ☐ 6. Build and run the Ch 9 VM interpreter

```bash
cd ch9
make
java j0 ../ch3/hello.java
# expected: hello, world
```

If `hello, world` prints, the interpreter path is complete.

---

## ☐ 7. (Optional) Build the Ch 13 x86-64 backend

Requires GCC or Clang on an x86-64 Linux or macOS machine.

```bash
cd ch13
make
./j0 ../ch3/hello.java    # compiles hello.java → a.out
./a.out
# expected: hello, world
```

---

## ☐ Final verification

```bash
cd ch9 && java j0 ../ch3/hello.java
# should print: hello, world
```

Full pipeline complete. See [roadmap](../../roadmap.md) for next steps.
