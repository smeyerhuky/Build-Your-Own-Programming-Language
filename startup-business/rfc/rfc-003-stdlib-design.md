---
type: RFC
title: "RFC-003 — J0 Standard Library Design"
description: Design specification for the J0 standard library including I/O, string, math, and collections modules.
tags: [rfc, stdlib, j0, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
rfc_number: "003"
status: Draft
author: Clinton L. Jeffery
---

# RFC-003 — J0 Standard Library Design

| Field | Value |
|---|---|
| RFC Number | 003 |
| Status | Draft |
| Author | Clinton L. Jeffery |
| Created | 2026-06-28 |
| Last updated | 2026-06-28 |

---

## Summary

J0 v1.0 has minimal built-in I/O support hardcoded into the compiler. This RFC proposes a formal **standard library (stdlib)** for J0 to be introduced in J0 v1.1, adding dedicated modules for I/O, strings, math, and basic collections.

---

## Motivation

The current J0 spec supports only `System.out.println` and `System.out.print` as hardcoded compiler special cases. This is sufficient for the course but insufficient for:

1. **Enterprise DSL builders** who need string manipulation, numeric formatting, and collection types.
2. **Platform exercises** that require richer output and input.
3. **LangCraft sandbox demos** that showcase J0 as a real language.

A stdlib also teaches an important concept: *how a language bootstraps its own runtime library*.

---

## Design principles

1. **Minimal surface area.** Each module exposes only what is needed. Prefer simplicity over completeness.
2. **Familiar names.** Where possible, use Java-like method names so J0 developers have a small learning curve.
3. **No implicit imports.** All stdlib classes must be explicitly referenced by name (e.g., `Math.abs(x)`). This keeps name resolution simple.
4. **Pure J0 where possible.** Stdlib methods that can be written in J0 are written in J0. Methods requiring native I/O are marked `native` and implemented in the compiler runtime.

---

## Modules

### 1. `IO` — Input/Output

```java
class IO {
    // Write to stdout
    public static void println(int x)      // prints integer + newline
    public static void println(boolean x)  // prints boolean + newline
    public static void println(String x)   // prints string + newline
    public static void print(int x)
    public static void print(boolean x)
    public static void print(String x)

    // Read from stdin
    public static int    readInt()          // reads one integer token
    public static String readLine()         // reads one line
}
```

`System.out.println` and `System.out.print` remain as aliases for backward compatibility. New code should use `IO.println` and `IO.print`.

---

### 2. `Str` — String operations

```java
class Str {
    public static int    length(String s)
    public static String concat(String a, String b)     // a + b
    public static String substring(String s, int from, int to)
    public static int    indexOf(String s, String sub)  // -1 if not found
    public static String toUpperCase(String s)
    public static String toLowerCase(String s)
    public static int    toInt(String s)                // parses integer; throws if invalid
    public static String fromInt(int n)                 // integer → string
    public static boolean equals(String a, String b)
}
```

*Rationale: J0 does not have operator overloading, so string concatenation cannot use `+`. `Str.concat` is the explicit form.*

---

### 3. `Math` — Numeric utilities

```java
class Math {
    public static int abs(int x)
    public static int max(int a, int b)
    public static int min(int a, int b)
    public static int pow(int base, int exp)  // integer exponentiation; exp >= 0
    public static int mod(int a, int b)       // always non-negative (unlike %)
    public static int clamp(int x, int lo, int hi)
}
```

*Floating-point support is deferred to RFC-007.*

---

### 4. `Array` — Array utilities

```java
class Array {
    public static int    length(int[] a)
    public static int    length(boolean[] a)
    public static void   fill(int[] a, int val)
    public static void   copy(int[] src, int[] dst, int n)  // copies first n elements
    public static int    sum(int[] a)
    public static int    max(int[] a)
    public static int    min(int[] a)
}
```

*J0 v1.0 does not support generic types. Separate overloads are provided per primitive type.*

---

### 5. `Collections` — Dynamic list (deferred to v1.2)

A basic `IntList` (dynamically-sized integer list) is planned but deferred to v1.2 to keep the stdlib small for the initial release:

```java
class IntList {
    public IntList()
    public void   add(int x)
    public int    get(int index)
    public int    size()
    public void   set(int index, int x)
    public void   remove(int index)
}
```

---

## Implementation approach

Stdlib classes are compiled to J0 bytecode and included in every `.j0` file at link time (similar to the JVM's `rt.jar`). The compiler pre-loads stdlib class definitions before processing user source.

`native` methods (those requiring JVM/OS calls for I/O) are resolved by the bytecode interpreter's native method dispatch table.

---

## Migration from v1.0

All `System.out.println(x)` calls in existing J0 programs remain valid. No changes to existing code are required. The compiler will emit a deprecation warning if a legacy `System.out` call is encountered.

---

## Drawbacks

- Adds compiler complexity (stdlib class loading, name lookup).
- `native` methods require parallel implementation in the Unicon and Java versions of the compiler runtime.
- Overloads for `IO.println` and `Array.length` require type-directed dispatch not currently in the type checker.

## Alternatives considered

- **Hardcode everything:** Already doing this; does not scale.
- **Use JVM reflection to expose Java stdlib:** Breaks the clean J0 abstraction; not compatible with Unicon runtime.

## Unresolved questions

- [ ] Should `Str` methods use 0-based or 1-based indices? (J0 arrays are 0-based; Java String is 0-based; Unicon strings are 1-based.) **Decision needed before implementation.**
- [ ] Should `IO.readLine()` return `null` on EOF or throw?
