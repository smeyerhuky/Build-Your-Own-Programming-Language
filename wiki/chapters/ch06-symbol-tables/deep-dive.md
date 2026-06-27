---
type: Chapter Section
title: "Ch 6 — Deep dive: symtab.java"
description: The HashMap + parent symbol table and its API.
resource: /ch6/symtab.java
tags: [deep-dive, symbol-table, hashmap, scope]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `symtab.java`

```java
public class symtab {
   String scope;
   symtab parent;
   HashMap<String,symtab_entry> t;

   symtab(String sc)             { scope = sc; t = new HashMap<>(); }
   symtab(String sc, symtab p)   { scope = sc; parent = p; t = new HashMap<>(); }

   symtab_entry lookup(String s) { return t.get(s); }

   void insert(String s, Boolean iC) {
      if (t.containsKey(s)) j0.semerror("redeclaration of " + s);
      else t.put(s, new symtab_entry(s, this, iC));
   }
   void insert(String s, Boolean iC, symtab sub) {
      if (t.containsKey(s)) j0.semerror("redeclaration of " + s);
      else { sub.parent = this; t.put(s, new symtab_entry(s, this, iC, sub)); }
   }

   void print() { print(0); }
   void print(int level) {
      for(int i=0;i<level;i++) System.out.print(" ");
      System.out.println(scope + " - " + t.size() + " symbols");
      for(symtab_entry se : t.values()) se.print(level+1);
   }
}
```

## Three observations

1. **`lookup` is local-only.** Walking the parent chain is the
   caller's job — `tree.java` has the canonical idiom `for (symtab s
   = this.st; s != null; s = s.parent) { ... }`.
2. **`insert` collides → `j0.semerror`.** This is by design: J0
   forbids re-declaration in the same scope.
3. **`insert(name, iC, sub)`** carries a *sub-scope*: a class entry
   carries the class body's symbol table, a method entry carries the
   method body's. That's how cross-scope lookup ("`obj.field`")
   eventually resolves.

## Cross-references

- [Concept: Symbol table](/wiki/concepts/symbol-table.md)
- [Concept: Type system](/wiki/concepts/type-system.md)
