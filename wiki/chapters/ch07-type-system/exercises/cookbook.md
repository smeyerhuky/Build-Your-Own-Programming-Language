---
type: Cookbook
title: "Ch 7 — Cookbook"
description: Canonical type-checking patterns.
resource: /ch7/typeinfo.java
tags: [cookbook, types, type-checking]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 7 types

## 1. Type an arithmetic node

### Prompt
"Compute the type of an `AddExpr+` AST node."

### Reference solution
In `tree.java::calctype()`:

```java
case "AddExpr+": {
   typeinfo l = kids.get(0).calctype();
   typeinfo r = kids.get(1).calctype();
   if (l.kind == DOUBLE || r.kind == DOUBLE) typ = TY_DOUBLE;
   else if (l.kind == INT && r.kind == INT)  typ = TY_INT;
   else if (l.kind == STRING && r.kind == STRING) typ = TY_STRING; // concat
   else j0.semerror("invalid + operands");
   return typ;
}
```

### Why this is canonical
- Numeric promotion (`int + double → double`) lives in `calctype`,
  not `checktype`.
- String concatenation is the only other `+` overload J0 allows.

---

## 2. Add a new primitive type

### Prompt
"Add a `byte` primitive."

### Reference solution
1. Add `BYTE` to the lexer (Ch 3 cookbook).
2. In grammar's `Type` rule: `Type: INT | BYTE | DOUBLE | ...`.
3. In `typeinfo`, add `TY_BYTE`.
4. In `calctype`, treat `BYTE` like `INT` for arithmetic but distinct
   for assignment.

### Why this is canonical
Each new primitive touches lexer, grammar, typeinfo, and the
typing rules — in that order.

---

## 3. Type-check a method call

### Prompt
"Check that the actuals to `foo(a, b)` match `foo`'s signature."

### Reference solution
In `assigntype("MethodCall")`:

```java
methodtype mt = (methodtype)kids.get(0).typ;
tree args = kids.get(1);
if (args.kids.size() != mt.parms.length) j0.semerror("arity mismatch");
for (int i = 0; i < mt.parms.length; i++) {
   args.kids.get(i).assigntype(mt.parms[i].typ);
   if (!compatible(args.kids.get(i).typ, mt.parms[i].typ))
      j0.semerror("arg " + i + " type mismatch");
}
typ = mt.ret;
```

### Why this is canonical
Method-call typing is a top-down operation (`assigntype`) because the
target type for each actual comes from the callee's signature.

---

## 4. Disallow `void` in expression context

### Prompt
"Reject `int x = foo();` when `foo` returns `void`."

### Reference solution
In `checktype` for an assignment, after `calctype` on the RHS:

```java
if (rhs.typ.kind == VOID) j0.semerror("void in expression context");
```

### Why this is canonical
`void` is a type that exists only in declarations, never as a value.
The check happens in `checktype` because it's a use-site error, not
a definition error.
