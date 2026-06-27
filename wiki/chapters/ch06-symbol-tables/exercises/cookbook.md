---
type: Cookbook
title: "Ch 6 — Cookbook"
description: Canonical patterns for scope management.
resource: /ch6/symtab.java
tags: [cookbook, symbol-table, scope]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 6 symbol tables

## 1. Open a new block scope

### Prompt
"Make `{ ... }` blocks introduce a fresh scope so inner `int x;`
shadows outer `int x;`."

### Reference solution
In `tree.java::mkSymTables()`, when entering a `Block` node:

```java
case "Block":
   this.st = new symtab("block", parent.st);
   for (tree k : kids) k.mkSymTables(this);
   break;
```

### Why this is canonical
- New scope = new `symtab` with the enclosing `st` as parent.
- Children recurse with `this` as their `parent`, picking up the new
  scope.

---

## 2. Parent-chain lookup

### Prompt
"Resolve an `IDENTIFIER` use to its declaration walking outward."

### Reference solution
```java
symtab_entry resolve(String name) {
   for (symtab s = this.st; s != null; s = s.parent) {
      symtab_entry e = s.lookup(name);
      if (e != null) return e;
   }
   j0.semerror("undefined: " + name);
   return null;
}
```

### Why this is canonical
`symtab.lookup` is local-only; the walk is the *caller's*
responsibility.

---

## 3. Install a built-in

### Prompt
"Add a built-in `Math.sqrt(double): double` so `Math.sqrt(2.0)`
type-checks."

### Reference solution
At driver startup, before walking the AST:

```java
symtab math = new symtab("Math", global);
math.insert("sqrt", false, new symtab("sqrt"));
global.insert("Math", false, math);
```

### Why this is canonical
Built-ins are just synthetic symtab entries installed before
traversal. The same shape works for the `System` built-in already in
`/ch6/j0.java`.

---

## 4. Detect redeclaration cleanly

### Prompt
"When the user declares the same name twice in the same scope, point
at *both* lines."

### Reference solution
Extend `symtab.insert` to carry a `tree` reference for the
declaring node:

```java
void insert(String s, Boolean iC, tree decl) {
   symtab_entry prev = t.get(s);
   if (prev != null)
      j0.semerror("redeclaration of " + s
                  + " (previously at line " + prev.decl.tok.line + ")");
   else t.put(s, new symtab_entry(s, this, iC, decl));
}
```

### Why this is canonical
Errors are most useful when they point at the *first* declaration,
not just the second. The symbol-table entry already knows where each
name came from.
