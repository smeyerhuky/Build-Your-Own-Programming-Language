---
type: Phase Section
title: "Phase 1 — Checklist"
description: Ordered install checklist for every tool in Phase 1, with per-OS commands and the verification command for each.
tags: [phase, install, checklist]
timestamp: 2026-06-27T00:00:00Z
---

# Checklist — Phase 1: Install tools

Work through each item top to bottom. Each item ends with a
**verify** command that must succeed before you move on.

---

## ☐ 1. JDK 11 or later

**macOS:**
```bash
brew install openjdk@17
# follow brew's instructions to add to PATH
```

**Debian / Ubuntu:**
```bash
sudo apt-get install openjdk-17-jdk
```

**Windows:** download from [adoptium.net](https://adoptium.net/).

**Verify:**
```bash
java -version    # openjdk 17.x.x (or 11+)
javac -version   # javac 17.x.x
```

---

## ☐ 2. JFlex

Download `jflex-1.9.1.tar.gz` from [jflex.de](https://jflex.de/),
extract, add `bin/` to `PATH`.

Or install via package manager if available:
```bash
# Homebrew:
brew install jflex
# Debian/Ubuntu (may be older version):
sudo apt-get install jflex
```

**Verify:**
```bash
jflex --version   # JFlex 1.x.x
```

---

## ☐ 3. BYacc (Java mode) or CUP

**BYacc — macOS:**
```bash
brew install byacc
```

**BYacc — Debian / Ubuntu:**
```bash
sudo apt-get install byacc
```

**CUP** (pure-Java alternative — [cup.sourceforge.net](http://www2.cs.tum.edu/projects/cup/)):
Download `java-cup-11b.jar` and add a wrapper script to your `PATH`.

**Verify:**
```bash
byacc --version    # byacc 20220114 (or similar)
# or: cup --version
```

---

## ☐ 4. GNU Make

**macOS:**
```bash
brew install make
# note: Homebrew installs as gmake; alias if needed
```

**Debian / Ubuntu:**
```bash
sudo apt-get install make
```

**Windows:** install via WSL, Chocolatey, or MSYS2.

**Verify:**
```bash
make --version    # GNU Make 4.x
```

---

## ☐ 5. GCC or Clang (for Ch 13 only)

**macOS:**
```bash
xcode-select --install    # installs clang
```

**Debian / Ubuntu:**
```bash
sudo apt-get install gcc
```

**Verify:**
```bash
gcc --version    # gcc 11.x or similar
# or: clang --version
```

---

## ☐ 6. Unicon (optional — Unicon path only)

Download from [unicon.sourceforge.net](http://unicon.sourceforge.net/)
or build from source:

```bash
git clone https://github.com/uniconproject/unicon.git
cd unicon && ./configure && make
```

**Verify:**
```bash
unicon --version
uflex --version
iyacc --version
```

---

## ☐ Final: all-in-one check

```bash
java -version && javac -version && jflex --version && byacc --version && make --version
```

All five lines must print version numbers. If any line fails,
revisit that tool's install step above.
