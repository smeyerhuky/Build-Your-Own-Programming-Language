---
type: Edge Case Catalog
title: "Phase 1 — Edge cases (negative examples)"
description: Common tool-install failures with CoT traces and fixes.
tags: [edge-cases, install, negative-examples, debugging]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Phase 1: Install tools

## 1. `jflex: command not found` after install

### Symptom
```
$ jflex --version
bash: jflex: command not found
```

### CoT trace
1. `jflex` is not on the `PATH`, even though the download succeeded.
2. JFlex ships as a `.tar.gz`; the binary is in `extracted/bin/jflex`.
3. That `bin/` directory was not added to `PATH`.
4. `which jflex` returns empty.

### Fix
Add the JFlex `bin/` directory to `PATH`:
```bash
export PATH="$PATH:/path/to/jflex-1.9.1/bin"
# add to ~/.bashrc or ~/.zshrc for persistence
```

Or create a wrapper script in a directory already on `PATH`:
```bash
cat > /usr/local/bin/jflex << 'EOF'
#!/bin/sh
exec java -jar /path/to/jflex-1.9.1/lib/jflex-1.9.1.jar "$@"
EOF
chmod +x /usr/local/bin/jflex
```

### Lesson
Package-managed installs (Homebrew, apt) handle PATH automatically.
Manual `.tar.gz` installs do not.

---

## 2. `byacc` installed but `-J` flag not recognized

### Symptom
```
$ byacc -J j0gram.y
byacc: illegal option -- J
```

### CoT trace
1. BYacc reports `illegal option` — the `-J` (Java output) flag does
   not exist in this version.
2. The `-J` flag was added in BYacc 20210808. Older system packages
   (common on Ubuntu 18/20) are pre-1.9 and lack it.
3. `byacc --version` shows a date before 2021.

### Fix
Build BYacc from source (Berkeley Yacc):
```bash
# download from invisible-island.net/byacc
./configure && make && sudo make install
```
Or switch to CUP (pure-Java, no version issues):
```bash
java -jar cup.jar j0gram.y
```

### Lesson
System package managers often ship outdated versions. For tools where
a specific flag is required, prefer a manual or Homebrew install over
`apt`.

---

## 3. `java -version` shows JRE, not JDK — `javac` missing

### Symptom
```
$ java -version
openjdk version "11.0.18"
$ javac -version
bash: javac: command not found
```

### CoT trace
1. `java` is the runtime (JRE), which may ship separately from the
   compiler (JDK).
2. `javac` is part of the JDK, not the JRE.
3. A JRE-only install is not sufficient for this course.

### Fix
Install the full JDK:
```bash
# Debian/Ubuntu:
sudo apt-get install openjdk-17-jdk
# macOS:
brew install openjdk@17
```

### Lesson
"Java" can mean the JRE, the JDK, or both. Always verify `javac` is
present, not just `java`.

---

## 4. Two JDK versions on the same machine, wrong one active

### Symptom
```
$ java -version    # 17
$ javac -version   # 11  (mismatch!)
```

### CoT trace
1. `java` resolves to `/usr/bin/java` → JDK 17.
2. `javac` resolves to `/usr/local/bin/javac` → JDK 11 installed earlier.
3. Class files compiled with JDK 17 can't run on JRE 11.

### Fix
Pin to a single JDK with `JAVA_HOME`:
```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)   # macOS
export PATH="$JAVA_HOME/bin:$PATH"
```

### Lesson
When multiple JDK versions coexist, `JAVA_HOME` is the authoritative
override. Set it explicitly in your shell profile.
