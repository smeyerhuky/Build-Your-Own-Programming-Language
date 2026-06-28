---
type: Phase Section
title: "Phase 1 — Deep dive: tool internals"
description: Under-the-hood detail on how each tool works and how they interact, for when install steps succeed but the integration fails.
tags: [phase, install, deep-dive]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive — Phase 1: Tool internals

## JDK: compiler + runtime together

`javac` (the compiler) and `java` (the runtime) are both part of the
JDK. An older JRE (runtime-only) install is not sufficient — `javac`
must be present.

```
JDK/
├── bin/
│   ├── java       ← runs .class files
│   ├── javac      ← compiles .java → .class
│   └── jar        ← bundles .class into .jar
└── lib/
    └── rt.jar     (or modules/ in JDK 9+)
```

On macOS, `brew install openjdk@17` installs under `/usr/local/opt/openjdk@17/`.
Homebrew prints the exact `PATH` and `JAVA_HOME` instructions after
install — follow them.

## JFlex: NFA → DFA → Java

JFlex is a pure-Java tool shipped as a single `.jar`. Its internal steps:

1. Parse the `.l` file into a list of `(regex, action)` pairs.
2. Build an NFA from the regexes via Thompson's construction.
3. Subset-construct a DFA.
4. Minimize the DFA.
5. Emit `Yylex.java` with the DFA encoded as a transition table.

The `bin/jflex` wrapper is just:
```bash
exec java -jar "$JFLEX_HOME/lib/jflex-1.9.1.jar" "$@"
```

If the `jflex` command fails but `java` works, run the jar directly
to confirm:
```bash
java -jar /path/to/jflex.jar --version
```

## BYacc -J: LALR(1) → Java

BYacc generates an LALR(1) parser. The `-J` flag (Java mode) emits
a `parser.java` that contains the parse table as integer arrays and
a `yyparse()` method.

```
j0gram.y  ──[byacc -J]──▶  parser.java
                            ├── token constants (parser.IF, parser.DO, …)
                            └── yyparse() method
```

The token constants are what `javalex.l` imports — that is why both
files share the `parser.` prefix.

## Make: just a dependency runner

`make` reads a `makefile`, checks timestamps, and runs only the
commands whose output is older than its inputs. The chapter makefiles
encode: "if `javalex.l` is newer than `Yylex.java`, re-run JFlex".

Nothing in the course requires GNU Make specifically — any POSIX make
works. On Windows, WSL's `make` is the easiest path.

## GCC / Clang: assemble and link

Ch 13 emits AT&T-syntax x86-64 assembly. Any modern GCC or Clang
on an x86-64 host can assemble and link it:

```bash
gcc -o hello hello.s          # assemble + link
```

Apple Silicon Macs (arm64) cannot natively assemble x86-64; use an
x86-64 Linux VM or a CI environment for Ch 13 on M-series hardware.
