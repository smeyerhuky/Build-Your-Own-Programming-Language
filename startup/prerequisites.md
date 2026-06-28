---
type: Chapter Section
title: Prerequisites
description: Every piece of software you need to install before building any chapter of the J0 compiler.
tags: [startup, prerequisites, install, java, jflex, unicon]
timestamp: 2026-06-27T00:00:00Z
---

# Prerequisites

Install **all** of the following before attempting the environment
setup or any chapter build. Exact version numbers are listed where
they matter; newer patch releases are generally fine.

## Required for every chapter (Java path)

| Tool | Minimum version | Purpose |
|---|---|---|
| **JDK** | 11 | Compile and run all `.java` source files. |
| **JFlex** | 1.8 | Generate `Yylex.java` from `javalex.l` (used in Ch 3 and every later chapter). |
| **Yacc / BYacc** or **CUP** | any | Generate the parser from `j0gram.y` (Ch 4). `byacc -J` emits Java (some builds use `-j`); CUP is the pure-Java alternative. |

### Installing JDK

**macOS:**
```bash
brew install openjdk@17
```

**Debian / Ubuntu:**
```bash
sudo apt-get install openjdk-17-jdk
```

**Windows:** download from [adoptium.net](https://adoptium.net/) and
run the installer.

Verify:
```bash
java -version   # expect: openjdk 17 (or 11+)
javac -version
```

### Installing JFlex

Download `jflex-1.9.1.tar.gz` (or later) from
[jflex.de](https://jflex.de/) and extract it. Add the `bin/` directory
to your `PATH`, **or** invoke it via its jar:

```bash
java -jar /path/to/jflex.jar javalex.l
```

The chapter makefiles expect a `jflex` command on the path. Create a
wrapper script if needed:

```bash
#!/bin/sh
exec java -jar /usr/local/lib/jflex.jar "$@"
```

Verify:
```bash
jflex --version   # expect: JFlex 1.9.x
```

### Installing BYacc (Java mode)

**macOS:**
```bash
brew install byacc
```

**Debian / Ubuntu:**
```bash
sudo apt-get install byacc
```

Use `byacc -J` to emit Java source. Alternatively, install
[CUP](http://www2.cs.tum.edu/projects/cup/) and use it in place of
Byacc where the chapter makefile allows.

Verify:
```bash
byacc --version
```

---

## Required for Unicon chapters (Unicon / Uflex path)

Some chapters ship both a Java version and a Unicon version. The Java
path is self-contained; you only need the Unicon toolchain if you want
to build the `.icn` variants.

| Tool | Minimum version | Purpose |
|---|---|---|
| **Unicon** | 13.1 | Interpret or compile `.icn` files. Ships `uflex` and `iyacc` bundled. |

### Installing Unicon

Download the installer from [unicon.sourceforge.net](http://unicon.sourceforge.net/). On Linux you can also build from source:

```bash
git clone https://github.com/uniconproject/unicon.git
cd unicon
./configure && make
```

Verify:
```bash
unicon --version      # expect: Unicon 13.x
uflex --version       # bundled scanner generator
iyacc --version       # bundled parser generator
```

---

## Optional but recommended

| Tool | Purpose |
|---|---|
| **GNU Make** | All chapter directories ship a `makefile`. `make` automates the build steps. |
| **GCC / Clang** | Required to assemble and link the x86-64 output of Ch 13. |
| **Git** | Clone the repo and track your own changes. |

### Checking `make`

```bash
make --version   # GNU Make 4.x on Linux/macOS, nmake or WSL on Windows
```

### Checking a C compiler (for Ch 13)

```bash
gcc --version    # or: clang --version
```

---

## Quick checklist

Run these commands in a fresh terminal before proceeding to
[Environment setup](./environment-setup.md). Every line should print
a version number, not an error.

```bash
java -version
javac -version
jflex --version
byacc --version   # or: cup --version
make --version
# Unicon path only:
unicon --version
```
