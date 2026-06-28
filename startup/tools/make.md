---
type: Setup Tool
title: Make
description: GNU Make — automates the multi-step build in each chapter directory.
tags: [tool, make, build, automation]
timestamp: 2026-06-27T00:00:00Z
---

# Make

GNU Make reads a `makefile` and runs only the commands whose outputs
are older than their inputs. Each chapter ships a `makefile` that
encodes the correct build order for that chapter.

## Install

| Platform | Command |
|---|---|
| macOS | `xcode-select --install` (includes make), or `brew install make` |
| Debian / Ubuntu | `sudo apt-get install make` |
| Windows | WSL2 (`apt install make`), or MSYS2, or Chocolatey |

On macOS Homebrew installs as `gmake`; create an alias if needed:
```bash
alias make=gmake    # add to ~/.bashrc or ~/.zshrc
```

## Verify

```bash
make --version    # GNU Make 4.x
```

## Used by

Every chapter — `make` automates the `jflex` → `byacc` → `javac`
sequence in the correct order with dependency tracking.

## Key commands

```bash
make             # build the default target (usually 'all')
make clean       # remove generated and compiled files
make -n          # dry run: print commands without executing
make -B          # force rebuild of all targets
```

## Common errors

| Error | Fix |
|---|---|
| `No rule to make target 'all'` | You are in the wrong directory. `cd` into a chapter folder first. |
| `jflex: command not found` (inside make) | `jflex` is not on `PATH`. Add it and retry. |
| `makefile: No such file or directory` | The chapter uses `Makefile` (capital M) — check case. |
