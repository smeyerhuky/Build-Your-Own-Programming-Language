---
type: Setup Tool
title: JFlex
description: Lexer generator that compiles a .l spec into Yylex.java. Required from Ch 3 onward.
tags: [tool, jflex, lexer, scanner]
timestamp: 2026-06-27T00:00:00Z
---

# JFlex

JFlex compiles a `.l` Flex specification into `Yylex.java`, a Java
scanner class. Required by every chapter that builds the compiler.

See also: [Wiki: Flex / Uflex](/wiki/tools/flex-uflex.md)

## Install

| Platform | Command |
|---|---|
| macOS | `brew install jflex` |
| Debian / Ubuntu | `sudo apt-get install jflex` (may be older) |
| Manual | Download from [jflex.de](https://jflex.de/), extract, add `bin/` to `PATH` |

The manual install is preferred to get a recent version:
```bash
# extract and create wrapper:
cat > /usr/local/bin/jflex << 'EOF'
#!/bin/sh
exec java -jar /opt/jflex/lib/jflex-1.9.1.jar "$@"
EOF
chmod +x /usr/local/bin/jflex
```

## Verify

```bash
jflex --version    # JFlex 1.x.x
```

## Used by

- Ch 3: `jflex javalex.l` → `Yylex.java`
- All later chapters that ship their own `javalex.l` (same command).

## Common errors

| Error | Fix |
|---|---|
| `jflex: command not found` | Add JFlex `bin/` to `PATH` or create a wrapper script. |
| `Illegal character: <FEFF>` | BOM in `.l` file — remove with `sed -i '1s/^\xEF\xBB\xBF//' javalex.l` |
| `cannot find symbol: parser.DO` | `parser.java` is missing `DO` — regenerate it with BYacc first. |
