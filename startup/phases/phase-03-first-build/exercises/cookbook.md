---
type: Cookbook
title: "Phase 3 — Cookbook (positive examples)"
description: Canonical first-build patterns — generating the lexer, compiling, and running it against test inputs.
tags: [cookbook, first-build, positive-examples, lexer]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Phase 3: First build

## 1. The canonical three-step build

### Prompt
"What is the minimal sequence to build and run the Ch 3 lexer?"

### Reference solution
```bash
cd ch3
jflex javalex.l
javac Yylex.java simple.java token.java
java simple hello.java
```

### Why this is canonical
These three steps — generate, compile, run — are the template for
every chapter's build. Mastering them here makes Ch 4–13 builds
predictable.

---

## 2. Use `make` to automate the build

### Prompt
"How do I build Ch 3 with a single command?"

### Reference solution
```bash
cd ch3
make          # regenerates Yylex.java and compiles all Java files
java simple hello.java
```

### Why this is canonical
`make` handles dependency tracking: it re-runs JFlex only if
`javalex.l` has changed since `Yylex.java` was last generated. Use
`make` in every chapter to avoid stale-file bugs.

---

## 3. Test with a richer J0 file

### Prompt
"How do I lex a J0 program that exercises more of the language?"

### Reference solution
```bash
cat > /tmp/rich.java << 'EOF'
public class rich {
  public static void main(string[] argv) {
    int i = 0;
    while (i < 10) {
      i = i + 1;
    }
    System.out.println("done");
  }
}
EOF
java simple /tmp/rich.java
```

Expected: tokens for `while`, `<`, `10`, `+`, `1`, `"done"`, etc.

### Why this is canonical
This exercises keywords (`while`, `int`, `public`), comparison
operators (`<`), arithmetic (`+`), integer literals, and string
literals in a single test.

---

## 4. Count tokens to sanity-check

### Prompt
"How do I quickly check how many tokens a file produces?"

### Reference solution
```bash
java simple hello.java | wc -l
# hello.java should produce ~25 tokens
```

### Why this is canonical
A token count that is wildly off (e.g., 1 or 1000 instead of ~25)
usually signals a runaway rule (e.g., an unterminated string consuming
the rest of the file).
