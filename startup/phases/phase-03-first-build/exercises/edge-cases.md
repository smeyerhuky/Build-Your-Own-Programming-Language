---
type: Edge Case Catalog
title: "Phase 3 — Edge cases (negative examples)"
description: Common first-build failures with CoT traces and fixes.
tags: [edge-cases, first-build, negative-examples, debugging, lexer]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Phase 3: First build

## 1. `cannot find symbol: class Yylex`

### Symptom
```
$ javac simple.java
simple.java:3: error: cannot find symbol
   static Yylex yylexer;
          ^
symbol:   class Yylex
```

### CoT trace
1. `simple.java` imports `Yylex` by reference — there is no explicit
   `import` statement, it relies on both files being compiled together.
2. `Yylex.java` does not exist because `jflex javalex.l` has not been
   run yet.
3. The compile command only listed `simple.java`.

### Fix
Run JFlex first, then compile all three files together:
```bash
jflex javalex.l
javac Yylex.java simple.java token.java
```

### Lesson
JFlex must run before `javac`. `Yylex.java` is generated, not
checked-in; it won't exist on a fresh clone.

---

## 2. `java simple` exits immediately with no output

### Symptom
```
$ java simple
# (no output, returns to prompt immediately)
```

### CoT trace
1. `simple.java` checks `if (argv.length == 0)` and prints a usage
   line — but in this run, nothing was printed.
2. Actually, `simple` was invoked with no argument — the usage message
   should have printed, but something consumed it.
3. Or: `argv.length == 0` triggers `System.out.println("usage: …")`
   and the program exits without error — the usage was the output.

### Fix
Pass the filename as an argument:
```bash
java simple hello.java
```

### Lesson
`simple` requires exactly one argument: the path to a `.java` file.
Running it with no argument is not an error — it just prints usage
and exits.

---

## 3. Token numbers look wrong or unexpected

### Symptom
```
$ java simple hello.java
token 0: public
token 0: class
```

### CoT trace
1. Every token returns category `0` — the default Java `int` value.
2. `jflex javalex.l` was run, but the `parser.java` token constants
   are all `0` (or the class was compiled from a stale version).
3. BYacc was not run (or `parser.java` is missing), so `parser.IF`,
   `parser.CLASS`, etc. are all undefined — but `javac` would fail
   in that case.

### Fix
Check that `parser.java` contains non-zero constants:
```bash
grep 'BREAK\|CLASS\|IF' ch3/parser.java
```
If missing or all zero, the stub `ch3/parser.java` may need to be
regenerated. In Ch 3 specifically, `parser.java` is a hand-written
stub (not BYacc-generated) — ensure it is unmodified.

### Lesson
The `parser.java` constant values are the lexer–parser contract.
If they are wrong, the token numbers are wrong downstream.

---

## 4. JFlex produces warnings about unmatched input

### Symptom
```
Warning: There is a "catch-all" state in the generated DFA.
  Unmatched input in states: ...
```

### CoT trace
1. JFlex warns that some input character sequence may not be matched by
   any rule and will fall through to an implicit error action.
2. In `javalex.l` the catch-all is the `.` rule calling `j0.lexErr`.
   JFlex's warning is informational — the `.` rule is intentional.

### Fix
This warning is safe to ignore in `javalex.l` because the `.` rule is
the explicit catch-all. No action needed.

### Lesson
JFlex's catch-all warning is a reminder, not an error. It is expected
whenever a `.` rule is the last rule in the file.
