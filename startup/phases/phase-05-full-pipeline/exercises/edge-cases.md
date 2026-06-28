---
type: Edge Case Catalog
title: "Phase 5 — Edge cases (negative examples)"
description: Common full-pipeline failures with CoT traces and fixes.
tags: [edge-cases, full-pipeline, negative-examples, debugging, vm, native]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Phase 5: Full pipeline

## 1. Ch 4 parser reports syntax error on valid J0

### Buggy input
`hello.java` parses fine in Ch 3 but the Ch 4 parser reports:
```
syntax error at line 2
```

### CoT trace
1. Ch 4 parser is generated from `j0gram.y`. If the grammar was not
   regenerated after changes to `javalex.l`, the token constants in
   `parser.java` are stale.
2. `parser.CLASS` in the Ch 4 version may differ from `parser.CLASS`
   in the Ch 3 version if any `%token` was added in between.
3. A stale `Yylex.class` compiled against old `parser.class` returns
   incorrect token numbers → parser sees wrong tokens.

### Fix
```bash
cd ch4
byacc -J j0gram.y       # regenerate parser.java
jflex javalex.l          # regenerate Yylex.java against new parser.java
javac *.java             # recompile everything
java j0 ../ch3/hello.java
```

### Lesson
Every chapter that modifies `javalex.l` or `j0gram.y` must re-run
both BYacc and JFlex. Stale generated files produce confusing token
mismatches.

---

## 2. VM interpreter prints wrong output / infinite loop

### Symptom
```
$ cd ch9 && java j0 ../ch3/hello.java
hello, worldhello, worldhello, world...   (infinite)
```

### CoT trace
1. The `while` condition in the VM is never evaluated to false.
2. Check the TAC generation (Ch 8): the loop's branch-if-false
   instruction may use the wrong label or wrong comparison direction.
3. Check the VM loop dispatch (Ch 9): the program counter may not
   advance past a branch instruction.

### Fix
Add a TAC debug dump: run `java j0 -debug ../ch3/hello.java` (if the
edition supports it) to print each TAC instruction before execution.
Inspect the branch instructions for the loop.

### Lesson
Infinite loops in the VM almost always trace back to a wrong branch
in the TAC output. Dump the TAC before suspecting the interpreter.

---

## 3. Ch 13 assembles but `a.out` crashes with segfault

### Symptom
```
$ ./a.out
Segmentation fault (core dumped)
```

### CoT trace
1. The x86-64 output was assembled and linked but crashes at runtime.
2. Common cause: incorrect System V AMD64 calling convention — stack
   not aligned to 16 bytes before a `call` instruction.
3. Another common cause: `null` string pointer passed to `printf`.

### Fix
Check `x64.java`'s method-call emission. The `call` instruction
requires RSP to be aligned to 16 bytes. If the function prelude
pushes an odd number of registers, add a `sub $8, %rsp` / `add $8, %rsp`
pair to realign.

### Lesson
x86-64 System V ABI requires 16-byte stack alignment at every `call`
site. Getting this wrong produces segfaults only when the called
function uses SSE instructions (e.g., `printf`).

---

## 4. `make` in ch13 fails with `as: unrecognized option -arch x86_64`

### Symptom
```
$ cd ch13 && make
as: unrecognized option -arch x86_64
```

### CoT trace
1. The makefile or `x64.java` invokes `as -arch x86_64` — an
   Apple-specific flag.
2. On Linux, GNU `as` does not accept `-arch`.
3. On Apple Silicon (arm64), the flag is valid for `as` but produces
   code for x86_64 that cannot run natively.

### Fix
**Linux:** remove `-arch x86_64` from the assembler invocation.
**Apple Silicon:** use a Docker/VM running x86-64 Linux, or cross-compile
via `x86_64-linux-gnu-gcc`.

### Lesson
Ch 13's native backend is x86-64 specific. On ARM or non-Apple
systems, the assembler invocation may need adjustment.
