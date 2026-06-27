---
type: Edge Case Catalog
title: "Ch 6 — Edge cases"
description: Common scope-management bugs.
resource: /ch6/symtab.java
tags: [edge-cases, cot, symbol-table, scope]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 6 symbol tables

## 1. Local lookup mistaken for resolution

### Buggy input
A check pass uses `this.st.lookup(name)` directly:

```java
if (st.lookup(use_name) == null) j0.semerror("undefined: " + use_name);
```

### Symptom
Variables declared in enclosing scopes are reported as undefined.

### CoT trace
1. `symtab.lookup` only inspects the current scope's HashMap.
2. To find an outer-scope declaration, walk the `parent` chain.
3. Wrap the call in the canonical `for (s = st; s != null; s =
   s.parent)` loop.

### Fix
Use the parent-chain resolver from the cookbook.

### Lesson
The symbol-table API is intentionally narrow. Scope walking is the
caller's job, and forgetting it produces false negatives.

---

## 2. Block scope omitted

### Buggy input
`mkSymTables` doesn't create a new `symtab` for `Block`, so inner
declarations land in the enclosing method scope.

### Symptom
```
{ int x; }
{ int x; }
```
fails with `redeclaration of x` — the two blocks share one scope.

### CoT trace
1. Each `{ ... }` is supposed to be its own scope.
2. Without a new `symtab`, both `insert("x", …)` calls hit the same
   HashMap.
3. The HashMap's second `containsKey` triggers `semerror`.

### Fix
Add a `Block` case in `mkSymTables` that constructs a fresh
`symtab("block", parent.st)`.

### Lesson
Scope opening is a positional decision encoded in the traversal, not
in the symbol-table class itself.

---

## 3. Forward reference into an unopened scope

### Buggy input
A method declared *after* it's used:

```java
public class C {
   public static void caller() { callee(); }
   public static void callee() { }
}
```

### Symptom
`undefined: callee`.

### CoT trace
1. The traversal `populateSymTables` walks top-down, left-to-right.
2. When `caller`'s body is checked, `callee` hasn't been inserted yet
   in the class scope.
3. Fix by doing a **two-pass** pattern: first pass installs all
   method *headers* into the class scope; second pass walks the
   bodies.

### Fix
In `j0.java`, split traversal into `populateClassHeaders()` then
`populateMethodBodies()`.

### Lesson
Forward references in method-level languages require either a
two-pass traversal or a deferred-resolution mechanism. Choose
explicitly; don't let the single-pass walk dictate semantics.

---

## 4. Sub-scope's `parent` not wired

### Buggy input
Someone calls `symtab.insert(name, iC, sub)` but uses a different
`sub` constructor that doesn't set `sub.parent`.

### Symptom
Lookups inside the sub-scope can't see outer names.

### CoT trace
1. `insert(name, iC, sub)` is supposed to set `sub.parent = this`.
2. If the caller already passed in a `symtab` with a stale `parent`,
   the wiring breaks silently.
3. The canonical `symtab.insert` enforces this — calling the wrong
   constructor bypasses it.

### Fix
Always go through `symtab.insert(..., sub)`; never set `sub.parent`
by hand from the caller.

### Lesson
Encapsulate the "open a sub-scope" operation in `symtab.insert` so
the parent pointer is set in one place.
