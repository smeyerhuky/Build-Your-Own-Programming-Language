---
type: Cookbook
title: "Phase 2 — Cookbook (positive examples)"
description: Canonical patterns for cloning the repo and verifying tools against real source files.
tags: [cookbook, clone, verify, positive-examples]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Phase 2: Clone & verify

## 1. Clone and navigate to the correct directory

### Prompt
"How do I clone the repo and confirm I'm in the right directory?"

### Reference solution
```bash
git clone https://github.com/smeyerhuky/Build-Your-Own-Programming-Language.git
cd Build-Your-Own-Programming-Language
ls ch3/javalex.l    # should print the filename, not an error
```

### Why this is canonical
`ls ch3/javalex.l` is a minimal signal: if it exists, you're in the
right repo root and the source is present.

---

## 2. Verify JFlex on the real spec

### Prompt
"How do I confirm JFlex works on the actual lexer spec?"

### Reference solution
```bash
cd ch3
jflex javalex.l
ls Yylex.java    # should exist and be non-empty
```

### Why this is canonical
This is the earliest possible use of the real build artifact. Any
JFlex version incompatibility shows up here with a clear error.

---

## 3. Verify BYacc on the real grammar

### Prompt
"How do I confirm BYacc works on the actual grammar?"

### Reference solution
```bash
cd ch4
byacc -J j0gram.y
ls parser.java    # or the generated parser file name
```

### Why this is canonical
`j0gram.y` is the real grammar with all shift/reduce declarations.
Running BYacc on it exposes any grammar ambiguities BYacc flags on
first parse.

---

## 4. Confirm Make builds ch3 end-to-end

### Prompt
"How do I confirm the makefile in ch3 works from scratch?"

### Reference solution
```bash
cd ch3
make clean 2>/dev/null || true    # remove any stale artifacts
make
ls simple.class    # build succeeded
```

### Why this is canonical
`make` after `make clean` is the gold standard for confirming a fresh
build works. It exercises jflex, javac, and the makefile dependency
ordering together.
