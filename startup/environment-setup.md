---
type: Chapter Section
title: Environment Setup
description: Clone the repo, verify tools, and configure your editor so every chapter build works from the start.
tags: [startup, setup, environment, clone, make]
timestamp: 2026-06-27T00:00:00Z
---

# Environment setup

Complete [Prerequisites](./prerequisites.md) before this page.

## 1 — Clone the repository

```bash
git clone https://github.com/smeyerhuky/Build-Your-Own-Programming-Language.git
cd Build-Your-Own-Programming-Language
```

The repository layout:

```
Build-Your-Own-Programming-Language/
├── ch3/    ← lexer (scanner)
├── ch4/    ← parser
├── ch5/    ← AST builder
├── ch6/    ← symbol tables
├── ch7/    ← type checker
├── ch8/    ← TAC / intermediate-rep
├── ch9/    ← bytecode VM interpreter
├── ch11/   ← bytecode serialization
├── ch12/   ← bytecode code generator
├── ch13/   ← x86-64 native code generator
├── startup/  ← this guide
├── wiki/     ← deep-reference OKF bundle
└── README.md
```

Ch 1, 2, 10, 14–17 are book-only chapters; no source folder ships for
them.

---

## 2 — Smoke-test your Java toolchain

Run these two commands from the repo root. They should compile and
run without errors.

```bash
# Compile a tiny Java file
echo 'public class Smoke { public static void main(String[] a) { System.out.println("ok"); } }' \
  > /tmp/Smoke.java
javac /tmp/Smoke.java -d /tmp
java -cp /tmp Smoke
# expected output: ok
```

---

## 3 — Smoke-test JFlex

```bash
cd ch3
jflex javalex.l          # produces Yylex.java in the current directory
ls Yylex.java            # should exist
cd ..
```

If `jflex` is not on your `PATH`, replace it with the jar invocation
from [Prerequisites](./prerequisites.md).

---

## 4 — Smoke-test the parser generator

```bash
cd ch4
byacc -J j0gram.y        # produces parser.java (BYacc) or Parser.java (CUP)
ls *.java                # should include the generated parser file
cd ..
```

---

## 5 — Smoke-test Make

Every chapter directory ships a `makefile`. Run `make` with no target
to list the default build:

```bash
cd ch3
make
cd ..
```

If `make` is missing, install it via your package manager (see
[Prerequisites](./prerequisites.md)).

---

## 6 — (Unicon path only) Smoke-test Uflex

```bash
uflex ch3/javalex.l      # should produce javalex.icn
```

---

## Editor / IDE setup

The course uses standard Java and Unicon source files; any editor
works. Recommended setups:

| Editor | Tip |
|---|---|
| **VS Code** | Install the *Language Support for Java* extension (Red Hat). Open the root folder; it discovers all `ch*/` source trees. |
| **IntelliJ IDEA** | *File → Open* the root folder. Mark each `ch*/` as a source root. |
| **Vim / Neovim** | The repo is small; use `:set path=**` and `gf` to jump between files. |

No build-system plugins are required because each chapter is a
standalone `javac` + `make` project.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `jflex: command not found` | Add JFlex `bin/` to your `PATH`, or create a wrapper script (see Prerequisites). |
| `byacc: illegal option -- J` | Your `byacc` is too old (pre-1.9). Build from source or use CUP instead. |
| `make: No rule to make target 'all'` | You are in the wrong directory. `cd` into a chapter folder first. |
| `Yylex.java` missing after `jflex` | Check the JFlex output: if it printed errors, fix the `.l` file or upgrade JFlex. |
| Unicon `uflex` not found | Unicon may not be on your `PATH`. Run `which unicon` and check the `bin/` directory for `uflex`. |

---

Next: [First build](./first-build.md)
