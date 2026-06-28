---
type: Setup Tool
title: GCC / Clang
description: C compiler and linker needed to assemble and link the x86-64 output produced by Ch 13.
tags: [tool, gcc, clang, x86-64, native, ch13]
timestamp: 2026-06-27T00:00:00Z
---

# GCC / Clang

Ch 13 emits AT&T-syntax x86-64 assembly. GCC or Clang assembles it
into an object file and links it into a native executable.

GCC and Clang are interchangeable for this purpose; both accept the
same invocation:
```bash
gcc -o hello hello.s    # or: clang -o hello hello.s
```

## Install

| Platform | Command |
|---|---|
| macOS | `xcode-select --install` (installs Clang) |
| Debian / Ubuntu | `sudo apt-get install gcc` |
| Windows | Install WSL2 then `apt-get install gcc` inside WSL |

## Verify

```bash
gcc --version     # gcc 11.x or later
# or: clang --version
```

## Used by

- Ch 13 only: assemble and link the `.s` file produced by `x64.java`.

## Platform notes

| Platform | Status |
|---|---|
| x86-64 Linux | Fully supported. |
| x86-64 macOS | Fully supported with Xcode CLI tools. |
| Apple Silicon (arm64) | **Not supported natively.** Use a Docker/VM for x86-64. |
| Windows native | **Not supported.** Use WSL2 with x86-64 Linux. |

## Common errors

| Error | Fix |
|---|---|
| `as: unrecognized option -arch x86_64` | On Linux, remove the `-arch x86_64` flag from the makefile/invocation. |
| `Segmentation fault` at runtime | Stack not 16-byte aligned before `call`. See Phase 5 edge cases. |
| `undefined reference to printf` | Ensure `-lc` or use GCC (which links libc automatically). |
