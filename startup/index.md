---
type: Startup Bundle
title: "Startup in a Box — Build Your Own Programming Language"
description: A complete OKF onboarding bundle. Five phases walk a newcomer from zero tools to a fully working J0 compiler pipeline, with per-phase concept pages, tool cards, troubleshooting, and LLM-prompt patterns.
tags: [startup, onboarding, okf, startup-in-a-box]
timestamp: 2026-06-27T00:00:00Z
---

# Startup in a Box

This bundle is a self-contained **Open Knowledge Format** onboarding
suite for the *Build Your Own Programming Language* course. It is
designed so an LLM (or a human) can walk the folder tree, load only
what they need, and get from zero to a running J0 compiler without
reading the whole book first.

## How to read this bundle

1. Start here.
2. Open [`phases/index.md`](./phases/index.md) — five phases, each a
   chapter-equivalent folder with overview, checklist, deep-dive,
   concepts, and exercises.
3. Per-phase exercises split into `cookbook.md` (positive patterns) and
   `edge-cases.md` (negative / debugging).
4. Cross-phase setup concepts (what J0 is, what a token stream is, how
   the build system works) live under [`concepts/`](./concepts/index.md).
5. Tool quick-ref cards (install, verify, common errors) live under
   [`tools/`](./tools/index.md).
6. LLM-prompt patterns for onboarding tasks live under
   [`prompting/`](./prompting/index.md).

## OKF types defined by this bundle

| Type | Meaning |
|---|---|
| `Startup Bundle` | The bundle root (this file). |
| `Index` | A directory `index.md` for navigation. |
| `Phase` | A phase `index.md` — the startup equivalent of a chapter. |
| `Phase Section` | `overview.md`, `checklist.md`, `deep-dive.md`, `concepts.md`. |
| `Setup Concept` | A cross-phase concept page in `/startup/concepts/`. |
| `Setup Tool` | A tool quick-ref card in `/startup/tools/`. |
| `Setup Prompt Pattern` | An LLM-prompt template in `/startup/prompting/`. |
| `Cookbook` | Per-phase positive examples (what a correct setup looks like). |
| `Edge Case Catalog` | Per-phase negative examples (common failures and fixes). |
| `Log` | The bundle's `log.md`. |

## Quick-start paths

**Zero to token stream (30 min):**
Phase 1 → Phase 2 → Phase 3.

**Zero to running interpreter (2 hrs):**
All five phases, then follow the [roadmap](./roadmap.md) into Ch 4–9.

**LLM-assisted setup:**
Load `phases/index.md` + `prompting/setup-assistant.md`, then
prompt your LLM with a phase description and your error message.

## Existing quick-start pages

The following pages from the initial startup guide remain valid
entry points and are linked from the phase folders:

| Page | What it covers |
|---|---|
| [Prerequisites](./prerequisites.md) | All software you must install. |
| [Environment setup](./environment-setup.md) | Clone the repo and verify tools. |
| [First build](./first-build.md) | Run the Ch 3 lexer against `hello.java`. |
| [Roadmap](./roadmap.md) | Learning path through all 17 chapters. |

## What you will build

The course walks you through a complete compiler for **J0** — a small
Java subset. By the end you will have:

- a **Flex / Uflex lexer** that turns source text into tokens (Ch 3),
- a **Yacc / Iyacc parser** that turns tokens into a parse tree (Ch 4),
- an **AST** with symbol tables and type annotations (Ch 5–7),
- a **three-address-code** middle-end and a stack-VM interpreter (Ch 8–9),
- a **bytecode serializer/loader** (Ch 11–12),
- a **native x86-64 code generator** (Ch 13).

The full pipeline at a glance:

```
source.j0
   │  Ch 3  Lexer        → tokens
   │  Ch 4  Parser       → parse tree
   │  Ch 5  AST builder  → AST
   │  Ch 6  Symbol table → annotated AST
   │  Ch 7  Type checker → fully annotated AST
   │  Ch 8  TAC gen      → three-address code
   ├──▶ Ch 9  Stack VM   (executes TAC directly)
   ├──▶ Ch 11/12  Bytecode (.j0 file + interpreter)
   └──▶ Ch 13  x86-64    (native binary)
```

## Relationship to the wiki

The [`/wiki/`](/wiki/index.md) folder is a deep-reference OKF bundle —
concepts, per-chapter notes, LLM-prompting templates. Once you have
your environment working, the wiki is the place to look up *why*
something works the way it does.

This `startup/` folder is deliberately narrower: it only tells you
*what to run*.
