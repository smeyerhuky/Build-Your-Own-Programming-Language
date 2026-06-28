---
type: Setup Tool
title: BYacc / CUP
description: Parser generators that compile a .y grammar into Java source. Required from Ch 4 onward.
tags: [tool, byacc, cup, parser, grammar]
timestamp: 2026-06-27T00:00:00Z
---

# BYacc / CUP

Two alternatives exist for generating the J0 parser from `j0gram.y`:

| Tool | Flag | Output file |
|---|---|---|
| **BYacc** (Berkeley Yacc) | `byacc -J j0gram.y` | `parser.java` |
| **CUP** (Constructor of Useful Parsers) | `java -jar cup.jar j0gram.y` | `parser.java` + `sym.java` |

The course makefiles default to BYacc. Either tool can be made to work.

See also: [Wiki: Yacc / Iyacc](/wiki/tools/yacc-iyacc.md)

## Install BYacc

| Platform | Command |
|---|---|
| macOS | `brew install byacc` |
| Debian / Ubuntu | `sudo apt-get install byacc` (may be old; see note below) |
| Source | Download from [invisible-island.net/byacc](https://invisible-island.net/byacc/) |

**Note:** Ubuntu packages prior to 22.04 ship BYacc < 20210808 which
lacks the `-J` flag. Install from source or use CUP on older systems.

## Install CUP

Download `java-cup-11b.jar` from
[www2.cs.tum.edu/projects/cup](http://www2.cs.tum.edu/projects/cup/).
Create a wrapper:
```bash
cat > /usr/local/bin/cup << 'EOF'
#!/bin/sh
exec java -jar /opt/cup/java-cup-11b.jar "$@"
EOF
chmod +x /usr/local/bin/cup
```

## Verify

```bash
byacc --version    # byacc YYYYMMDD (expect 20210808 or later)
# or: cup --version
```

## Used by

- Ch 4+: regenerates `parser.java` (token constants + LALR(1) parse table).

## Common errors

| Error | Fix |
|---|---|
| `illegal option -- J` | BYacc too old. Build from source or switch to CUP. |
| `y.tab.c` produced instead of `parser.java` | Wrong flag (`-j` vs `-J`). Check `byacc -h`. |
| `1 shift/reduce conflict` (warning) | Usually safe. Check `y.output` with `byacc -v`. |
