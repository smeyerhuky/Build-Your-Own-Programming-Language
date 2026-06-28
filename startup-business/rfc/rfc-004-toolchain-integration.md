---
type: RFC
title: "RFC-004 — J0 Toolchain IDE/LSP Integration"
description: Specification for the J0 language server (LSP) and VS Code extension, enabling hover docs, diagnostics, go-to-definition, and code formatting for J0.
tags: [rfc, lsp, vscode, ide, j0, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
rfc_number: "004"
status: Draft
author: Clinton L. Jeffery
---

# RFC-004 — J0 Toolchain IDE/LSP Integration

| Field | Value |
|---|---|
| RFC Number | 004 |
| Status | Draft |
| Author | Clinton L. Jeffery |
| Created | 2026-06-28 |
| Last updated | 2026-06-28 |

---

## Summary

This RFC specifies the **J0 Language Server** (a Language Server Protocol server for J0) and the **LangCraft VS Code Extension** that wraps it. The language server enables editor features — hover documentation, live diagnostics, go-to-definition, and code formatting — for `.j0` source files.

---

## Motivation

The current J0 toolchain is command-line only. Developers write J0 in plain text editors with no language intelligence. Adding LSP support will:

1. **Reduce the error-feedback loop** from compile-run to keystroke.
2. **Improve the enterprise product** — teams building DSLs need IDE tooling.
3. **Differentiate LangCraft** from competitors that have no interactive tooling.
4. **Teach LSP integration** as a concept — the J0 language server is itself a teaching artifact.

---

## LSP version

This RFC targets **LSP 3.17** (the current stable version as of 2026).

---

## Architecture

```
VS Code Extension (TypeScript)
    │
    │  stdio / TCP
    │
J0 Language Server (Java)
    │
    ├── J0 Lexer (JFlex-generated)
    ├── J0 Parser (BYacc/Cup-generated)
    ├── J0 Symbol Table builder
    ├── J0 Type Checker
    └── J0 Formatter
```

The language server is a standalone Java process. The VS Code extension spawns it on activation and communicates over stdio using JSON-RPC (LSP protocol). The same server can be used by any LSP-compatible editor (Neovim, Emacs, IntelliJ).

---

## Supported LSP capabilities

### Phase 1 (v1.0 — P1 roadmap item)

| Capability | LSP Method | Description |
|---|---|---|
| Syntax highlighting | TextMate grammar (not LSP) | Static grammar-based highlighting |
| Diagnostics | `textDocument/publishDiagnostics` | Lexer and parser errors as squiggles |
| Hover | `textDocument/hover` | Type of expression; documentation for keywords |
| Go-to-definition | `textDocument/definition` | Jump to declaration of variable, method, or class |
| Document symbols | `textDocument/documentSymbol` | Class/method outline in the editor sidebar |

### Phase 2 (v1.1 — later roadmap item)

| Capability | LSP Method | Description |
|---|---|---|
| Completion | `textDocument/completion` | Method and field name completion |
| Signature help | `textDocument/signatureHelp` | Method parameter hints as you type |
| References | `textDocument/references` | Find all usages of a symbol |
| Rename | `textDocument/rename` | Safe rename across the file |
| Formatting | `textDocument/formatting` | Auto-format to J0 style guide |
| Code actions | `textDocument/codeAction` | Quick fixes for common errors (undefined variable, type mismatch) |

---

## VS Code extension specification

### Extension metadata

| Field | Value |
|---|---|
| Extension ID | `langcraft.j0-language` |
| Display name | J0 Language Support |
| Publisher | `langcraft` |
| VS Code engine | `^1.85.0` |
| Repository | `github.com/smeyerhuky/Build-Your-Own-Programming-Language` (extensions/ subfolder) |

### Activation events

```json
"activationEvents": [
    "onLanguage:j0",
    "workspaceContains:**/*.j0"
]
```

### Contributes

```json
"contributes": {
    "languages": [{
        "id": "j0",
        "aliases": ["J0", "j0"],
        "extensions": [".j0"],
        "configuration": "./language-configuration.json"
    }],
    "grammars": [{
        "language": "j0",
        "scopeName": "source.j0",
        "path": "./syntaxes/j0.tmLanguage.json"
    }],
    "commands": [{
        "command": "langcraft.runInSandbox",
        "title": "Run in LangCraft Sandbox",
        "category": "LangCraft"
    }, {
        "command": "langcraft.showTokenStream",
        "title": "Show Token Stream",
        "category": "LangCraft"
    }],
    "configuration": {
        "title": "LangCraft J0",
        "properties": {
            "langcraft.sandboxApiKey": {
                "type": "string",
                "description": "Your LangCraft API key (from langcraft.io/settings)"
            },
            "langcraft.javaPath": {
                "type": "string",
                "description": "Path to Java executable (defaults to JAVA_HOME)"
            }
        }
    }
}
```

---

## Language server startup protocol

1. VS Code extension activates on `.j0` file open.
2. Extension checks for `langcraft-ls.jar` in the extension bundle; downloads if missing.
3. Extension spawns: `java -jar langcraft-ls.jar --stdio`
4. LSP handshake: `initialize` → `initialized` → ready.
5. On file open/change: server re-lexes and re-parses; publishes diagnostics.
6. All LSP requests are answered within 500 ms (target; type-check is cached).

---

## Error reporting format

Diagnostics use the LSP `Diagnostic` structure:

```json
{
    "range": {
        "start": {"line": 4, "character": 8},
        "end":   {"line": 4, "character": 15}
    },
    "severity": 1,
    "code": "J0-E042",
    "source": "j0-language-server",
    "message": "Type mismatch: expected 'int', found 'boolean'",
    "relatedInformation": [{
        "location": {"uri": "...", "range": {...}},
        "message": "Variable 'x' declared as 'boolean' here"
    }]
}
```

Error codes follow the scheme `J0-E` + 3 digits (errors) and `J0-W` + 3 digits (warnings).

---

## Hover documentation

Hovering over a J0 keyword returns a markdown string with:
- Syntax description
- Brief semantics
- Link to the relevant OKF concept page in the LangCraft platform

Hovering over a user-defined identifier returns:
- Declared type
- Declaration site (file:line)

---

## Drawbacks

- Java process startup is slow (~500 ms first launch). Mitigate with keep-alive and incremental re-parsing.
- Distributing a JAR in the extension bundle increases extension size (~15 MB). Acceptable trade-off.

## Alternatives considered

- **Implement language server in TypeScript/Node:** Would avoid JVM startup cost but requires rewriting the J0 compiler in TypeScript — not worth it.
- **Use tree-sitter for parsing:** Good for syntax highlighting; insufficient for type-aware features.

## Unresolved questions

- [ ] Should the language server use stdio or TCP transport? (Stdio is simpler for VS Code; TCP enables multi-editor support.) **Propose stdio for v1, TCP as an option in v1.1.**
- [ ] How do we handle LSP requests during a long type-check? Debounce or cancel-and-restart?
