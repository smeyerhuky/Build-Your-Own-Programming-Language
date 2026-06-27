---
type: Tool
title: Unicon
description: The author's own descendant of Icon; parallel implementation language for every chapter.
tags: [unicon, icon, goal-directed]
timestamp: 2026-06-27T00:00:00Z
---

# Unicon

[Unicon](https://unicon.sourceforge.io/) is a high-level descendant
of Icon, co-designed by the book's author. It is goal-directed,
string-rich, and dynamically typed — almost the opposite of J0.

## Why the dual implementation

The book showcases two very different host languages compiling the
same source language. The Unicon code is typically 1.2–1.5× shorter
than the Java code because of Unicon's expression-based control flow
and built-in sequence types.

## Build

Each chapter has a `makefile` Unicon target that runs `unicon -s
*.icn` after Uflex/Iyacc regenerate the lexer/parser.

## File-naming pattern

Every Java file `foo.java` has a sibling `foo.icn`. Grammar tables in
Unicon are emitted as `j0gram.icn` and `j0gram_tab.icn`.
