---
type: Setup Tool
title: JDK
description: Java Development Kit — the compiler (javac) and runtime (java) needed by every chapter.
tags: [tool, jdk, java, javac]
timestamp: 2026-06-27T00:00:00Z
---

# JDK

The JDK provides `javac` (compiler) and `java` (runtime). Both must be
from the same version. JDK 11 is the minimum; JDK 17 is recommended.

## Install

| Platform | Command |
|---|---|
| macOS | `brew install openjdk@17` |
| Debian / Ubuntu | `sudo apt-get install openjdk-17-jdk` |
| Windows | Download from [adoptium.net](https://adoptium.net/) |

## Verify

```bash
java -version    # openjdk version "17.x.x"
javac -version   # javac 17.x.x  (must match java major version)
```

## Used by

Every chapter — `javac` compiles all `.java` source files; `java`
runs the compiler and (in Ch 9) runs the VM interpreter.

## Common errors

| Error | Fix |
|---|---|
| `javac: command not found` | You have the JRE only. Install the full JDK. |
| `java` and `javac` version mismatch | Set `JAVA_HOME` to a single JDK install. |
| `UnsupportedClassVersionError` at runtime | Source was compiled with a newer JDK than the runtime. Use the same JDK for both. |
