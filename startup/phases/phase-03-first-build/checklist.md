---
type: Phase Section
title: "Phase 3 — Checklist"
description: Step-by-step build checklist for Phase 3 — jflex → javac → java simple → token stream.
tags: [phase, first-build, checklist]
timestamp: 2026-06-27T00:00:00Z
---

# Checklist — Phase 3: First build

---

## ☐ 1. Enter the ch3 directory

```bash
cd ch3
```

---

## ☐ 2. Generate the lexer

```bash
jflex javalex.l
```

**Expected output snippet:**
```
Reading "javalex.l"
Constructing NFA : 107 states in NFA
Converting NFA to DFA : ...
Writing code to "Yylex.java"
```

**Verify:**
```bash
ls Yylex.java    # must exist
```

---

## ☐ 3. Compile the scanner + driver

```bash
javac Yylex.java simple.java token.java
```

**Verify:**
```bash
ls simple.class token.class Yylex.class
```

No compilation errors expected. If you see `cannot find symbol` or
`package not found`, check [edge cases](./exercises/edge-cases.md).

---

## ☐ 4. Run the driver against hello.java

```bash
java simple hello.java
```

**Expected output (abbreviated):**
```
token 258: public
token 258: class
token 258: hello
token 273: {
...
token 295: "hello, world"
...
```

The exact token numbers depend on the `parser.java` constants. The
important thing: every keyword, identifier, string literal, and
punctuation mark in `hello.java` should appear.

---

## ☐ 5. Run against your own test file

```bash
cat > /tmp/mytest.java << 'EOF'
public class mytest {
  public static void main(string[] argv) {
    int x = 42;
    System.out.println("ok");
  }
}
EOF
java simple /tmp/mytest.java
```

**Verify:** `int`, `x`, `=`, `42`, and `"ok"` all appear as separate tokens.

---

## ☐ Shortcut: use make

```bash
make simple    # regenerates Yylex.java and compiles in one step
java simple hello.java
```

---

All checks pass → proceed to [Phase 4](../phase-04-extend-j0/index.md)
or read the [roadmap](../../roadmap.md) to continue through Ch 4–9.
