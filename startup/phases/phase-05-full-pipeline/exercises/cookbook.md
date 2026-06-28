---
type: Cookbook
title: "Phase 5 — Cookbook (positive examples)"
description: Canonical full-pipeline build patterns — building each chapter layer and testing end-to-end.
tags: [cookbook, full-pipeline, positive-examples, vm, native]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Phase 5: Full pipeline

## 1. Build and test each chapter in sequence

### Prompt
"What is the canonical build-and-test command for a chapter?"

### Reference solution
```bash
cd chN      # replace N with chapter number (4, 5, 6, 7, 8, or 9)
make
java j0 ../ch3/hello.java
```

Each chapter's `j0` class adds a new compiler phase. Ch 4 parses,
Ch 5 builds an AST, …, Ch 9 executes.

### Why this is canonical
`hello.java` is the universal test input. Every chapter should
process it without errors. A failure in `chN` that did not occur in
`ch(N-1)` is localized to the new phase added in chapter N.

---

## 2. Test with a program that exercises control flow

### Prompt
"How do I test that the VM executes a loop correctly?"

### Reference solution
```bash
cat > /tmp/loop.java << 'EOF'
public class loop {
  public static void main(string[] argv) {
    int i = 0;
    while (i < 3) {
      System.out.println("i");
      i = i + 1;
    }
  }
}
EOF
cd ch9 && java j0 /tmp/loop.java
# expected: "i" printed 3 times
```

### Why this is canonical
A `while` loop with a comparison and an update exercises the branch,
comparison, and arithmetic opcodes. It's the simplest non-trivial
control-flow test.

---

## 3. Run the bytecode path (Ch 11–12)

### Prompt
"How do I compile hello.java to a .j0 bytecode file and run it?"

### Reference solution
```bash
cd ch12
make
java ch12.j0 ../ch3/hello.java     # generates ../ch3/hello.j0
cd ../ch11 && make && java j0x ../ch3/hello.j0
# expected: hello, world
```

### Why this is canonical
The `-bc` flag activates the bytecode code-generator path. Running
the `.j0` file via the Ch 11 interpreter validates both the emitter
(Ch 12) and the loader/interpreter (Ch 11).

---

## 4. Build and run the x86-64 native binary (Ch 13)

### Prompt
"How do I compile hello.java to a native binary and run it?"

### Reference solution
```bash
cd ch13
make
./j0 ../ch3/hello.java    # emits hello.s + assembles to a.out
./a.out
# expected: hello, world
```

Requires an x86-64 machine with GCC or Clang in PATH.

### Why this is canonical
`./j0` runs the J0 compiler (the native-codegen version); `./a.out`
runs the resulting native binary. These are the two executables Ch 13
produces.
