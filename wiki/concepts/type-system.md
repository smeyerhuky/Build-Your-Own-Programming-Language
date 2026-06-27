---
type: Concept
title: Type system
description: Primitive, array, class, and method types, with compatibility rules used during type checking.
tags: [types, type-checking, typeinfo, classtype, methodtype]
timestamp: 2026-06-27T00:00:00Z
---

# Type system

The J0 type system is small and explicit:

- Primitives: `int`, `double`, `bool`, `string`, `void`.
- Arrays: any type plus one or more `[]` dimensions.
- Classes: user-declared classes with fields and methods.
- Methods: a return type plus an ordered list of `Parameter`s.

Type information lives on every AST node as a `typeinfo` reference,
attached during a Ch 7 traversal.

## Where it appears in the book

- Introduced in [Ch 7 — Checking Base Types](/wiki/chapters/ch07-type-system/index.md).
- Refined in Ch 8 for arrays, method calls and struct accesses.

## Anchor files

- `/ch7/typeinfo.java` — primitive descriptor.
- `/ch7/arraytype.java` — wraps a base type plus dimension count.
- `/ch7/classtype.java` — class name + symbol table of fields/methods.
- `/ch7/methodtype.java` — return type + `Parameter[]`.
- `/ch7/parameter.java` — a single formal parameter (name + type).
- `/ch7/tree.java::calctype()` / `checktype()` / `assigntype()` — the
  three AST passes that compute, validate, and propagate types.

## Compatibility rules

- Arithmetic operators require both operands numeric; `int + double`
  promotes to `double`.
- Logical operators require `bool`.
- Equality `==` / `!=` allows numeric ↔ numeric and class ↔ class
  (where the two classes are the same or one is `null`).
- Assignment requires LHS and RHS types match exactly (J0 does not do
  implicit narrowing).

## Common bugs

- Forgetting to set `typ` on a new AST node leaves later passes with a
  `NullPointerException` deep inside `calctype`.
- Silent int → double coercion is allowed *only* in arithmetic, not
  in assignment.

See [`/wiki/chapters/ch07-type-system/exercises/edge-cases.md`](/wiki/chapters/ch07-type-system/exercises/edge-cases.md).
