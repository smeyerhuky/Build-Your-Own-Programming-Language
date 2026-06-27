---
type: Edge Case Catalog
title: "Ch 7 — Edge cases"
description: Common type-system bugs.
resource: /ch7/typeinfo.java
tags: [edge-cases, cot, types, type-checking]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 7 types

## 1. Silent int → double in assignment

### Buggy input
```
int x;
x = 1.5;
```

### Symptom
With a buggy `checktype` that allows numeric promotion in
assignments, `x = 1.5` truncates silently to `1`.

### CoT trace
1. J0 allows int↔double promotion only in **expressions**, not
   assignments.
2. The check must be stricter at assignment sites.
3. Reject assignment when LHS is `int` and RHS is `double`.

### Fix
In `checktype("Assignment")`:

```java
if (lhs.typ.kind != rhs.typ.kind) j0.semerror("assignment type mismatch");
```

### Lesson
Promotion rules differ by context. Expression-level rules ("the
wider type wins") aren't the right default for assignment.

---

## 2. Array dimension mismatch

### Buggy input
```
int[][] m;
int   x;
m = x;
```

### Symptom
With a buggy comparator that only checks the element type, the
assignment is accepted. Later codegen produces garbage indexing.

### CoT trace
1. `arraytype` has `(elem, dims)`. Both must match.
2. The check needs `l.dims == r.dims` *in addition to* element-type
   equality.
3. A primitive (`dims == 0`) is never assignable to/from an array.

### Fix
In a `compatible(a, b)` helper:

```java
if (a instanceof arraytype && b instanceof arraytype)
   return ((arraytype)a).dims == ((arraytype)b).dims
       && compatible(((arraytype)a).elem, ((arraytype)b).elem);
return a.kind == b.kind && a.dims == 0 && b.dims == 0;
```

### Lesson
Type equality is structural. Forgetting any structural field gives a
permissive (incorrect) `compatible`.

---

## 3. `null` without a target type

### Buggy input
```
return null;     // in a method whose return type is int
```

### Symptom
`calctype` for `null` defaults to a class type, and `checktype` then
fails with a confusing "class ↔ int" message.

### CoT trace
1. `null` has no inherent type; it borrows from context.
2. The fix is to call `assigntype(targetReturnType)` on the `null`
   node before `checktype`.
3. `assigntype` can then reject `null` for primitive targets cleanly.

### Fix
Top-down propagation: `ReturnStmt.assigntype` pushes the enclosing
method's return type into the child expression.

### Lesson
Context-dependent values (`null`, array literals, ternary results)
need top-down typing in addition to bottom-up `calctype`.

---

## 4. Method overload collisions

### Buggy input
```
public static void f(int x) { }
public static void f(double x) { }
```

### Symptom
With a name-only symbol table, the second declaration triggers
`redeclaration of f`.

### CoT trace
1. J0's `symtab.insert` keys on name only — overloading would
   require keying on `(name, signature)`.
2. The course's scope is single-dispatch by name, so this is by
   design.
3. To support overloading, replace the per-class scope's HashMap
   with a `Map<String,List<methodtype>>` and resolve at call sites
   using actual-argument types.

### Fix (if you want overloading)
Sketch:

```java
class symtab_entry { List<methodtype> overloads; }
```
and disambiguate in `assigntype("MethodCall")`.

### Lesson
Symbol-table key shape is a *language-design* decision. Adding
overloading isn't a one-line patch — it changes lookup everywhere.
