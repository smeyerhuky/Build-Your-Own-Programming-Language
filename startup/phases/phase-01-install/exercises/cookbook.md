---
type: Cookbook
title: "Phase 1 — Cookbook (positive examples)"
description: Canonical patterns for verifying each tool install correctly. Each entry is a prompt + expected result pair.
tags: [cookbook, install, positive-examples]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Phase 1: Install tools

## 1. Verify JDK is installed and correct version

### Prompt
"How do I confirm my JDK is installed and is version 11 or later?"

### Reference solution
```bash
java -version
```
Look for `openjdk version "11` (or higher). If you see `version "8"` or
`version "1.8"`, the JDK is too old; install JDK 17.

Also check `javac`:
```bash
javac -version    # must match java -version major number
```

### Why this is canonical
Both `java` and `javac` must be from the same JDK. A mismatch (e.g.,
runtime 17, compiler 11) causes subtle class-format errors at runtime.

---

## 2. Verify JFlex finds its jar

### Prompt
"How do I confirm JFlex is installed and can process a `.l` file?"

### Reference solution
```bash
jflex --version          # quick smoke test
echo '%%\n%%\n' | jflex -  # minimal .l input (may warn but not error)
```

### Why this is canonical
`jflex --version` only confirms the wrapper runs; feeding a minimal
spec confirms the jar is loadable and the tool works end-to-end.

---

## 3. Verify BYacc supports Java output

### Prompt
"How do I confirm my BYacc install can produce Java source?"

### Reference solution
```bash
byacc --version          # confirm tool exists
echo '%% start: /* empty */ ;' > /tmp/empty.y
byacc -J /tmp/empty.y    # should produce parser.java in current dir
ls parser.java
```

### Why this is canonical
Old BYacc versions (pre-1.9) do not support `-J`. Generating from a
trivial grammar is the fastest confirmation.

---

## 4. Verify Make finds the chapter makefile

### Prompt
"How do I confirm Make is working in a chapter directory?"

### Reference solution
```bash
cd ch3
make --version            # confirm make exists
make -n                   # dry run: shows commands without running
```

`make -n` prints what would run without executing anything — safe to
use even before all tools are ready.

### Why this is canonical
`make --version` only confirms Make exists. `make -n` confirms the
`makefile` is valid and that Make can parse it.
