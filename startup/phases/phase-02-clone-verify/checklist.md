---
type: Phase Section
title: "Phase 2 — Checklist"
description: Ordered verification checklist for Phase 2 — clone the repo and smoke-test each tool against the real source files.
tags: [phase, clone, verify, checklist]
timestamp: 2026-06-27T00:00:00Z
---

# Checklist — Phase 2: Clone & verify

---

## ☐ 1. Clone the repository

```bash
git clone https://github.com/smeyerhuky/Build-Your-Own-Programming-Language.git
cd Build-Your-Own-Programming-Language
```

**Verify:**
```bash
ls ch3/javalex.l    # should exist
ls ch4/j0gram.y     # should exist
ls wiki/index.md    # should exist
```

---

## ☐ 2. Inspect the chapter layout

```bash
ls -d ch*/
# expected: ch3  ch4  ch5  ch6  ch7  ch8  ch9  ch11  ch12  ch13
```

No `ch10`, `ch14`–`ch17` — those are book-only chapters without source.

---

## ☐ 3. Run JFlex on the real lexer spec

```bash
cd ch3
jflex javalex.l
ls Yylex.java
cd ..
```

JFlex should print NFA/DFA stats and write `Yylex.java`. Any error
here means either the tool is misconfigured or the `.l` file has
encoding issues.

---

## ☐ 4. Run BYacc on the real grammar

```bash
cd ch4
byacc -J j0gram.y
ls *.java    # should include a parser .java file
cd ..
```

---

## ☐ 5. Confirm `make` works in ch3

```bash
cd ch3
make
cd ..
```

`make` reads the chapter's `makefile`, regenerates `Yylex.java`, and
compiles. A successful run ends with no error messages.

---

## ☐ 6. (Optional) Unicon path smoke-test

```bash
uflex ch3/javalex.l    # produces javalex.icn
```

---

## ☐ Final check

```bash
# ch3 build artifacts exist
ls ch3/Yylex.java ch3/simple.class 2>/dev/null || echo "run make in ch3 first"
```

All checks pass → proceed to [Phase 3](../phase-03-first-build/index.md).
