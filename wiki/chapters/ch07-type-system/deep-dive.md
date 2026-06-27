---
type: Chapter Section
title: "Ch 7 — Deep dive: type classes"
description: typeinfo, arraytype, classtype, methodtype, parameter.
resource: /ch7/typeinfo.java
tags: [deep-dive, types, typeinfo, classtype, methodtype]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: type classes

Five small Java classes carry all the type information in J0.

## `typeinfo` — primitives

A `typeinfo` instance describes a primitive (`int`, `double`, `bool`,
`string`, `void`) or wraps a richer subclass (`arraytype`,
`classtype`, `methodtype`).

## `arraytype extends typeinfo`

```
arraytype { typeinfo elem; int dims; }
```

`int[][]` is `arraytype(int, 2)`. Constructed once in `calctype` when
visiting a `Type` node followed by `[ ]` markers.

## `classtype extends typeinfo`

```
classtype { String name; symtab body; }
```

The `body` symbol table is the same one Ch 6 attached to the class
node. That's how `obj.field` resolves: the type of `obj` carries a
pointer to its fields.

## `methodtype extends typeinfo`

```
methodtype { typeinfo ret; Parameter[] parms; }
```

`assigntype` for a `MethodCall` site uses
`methodtype.parms` to type-check the actuals one by one and
`methodtype.ret` to type the call expression.

## `Parameter`

```
Parameter { String name; typeinfo typ; }
```

Used both in method signatures and as the formal-parameter portion
of a method scope.

## Three AST passes

| Pass | Purpose | Direction |
|---|---|---|
| `calctype()` | Compute each expression's natural type. | Bottom-up |
| `checktype()` | Validate operator-operand compatibility. | Bottom-up |
| `assigntype()` | Push a target type into context-dependent nodes (e.g., `null`, method actuals). | Top-down |

## Cross-references

- [Concept: Type system](/wiki/concepts/type-system.md)
- [Concept: Symbol table](/wiki/concepts/symbol-table.md)
