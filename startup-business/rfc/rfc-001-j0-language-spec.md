---
type: RFC
title: "RFC-001 — J0 Language Specification"
description: Formal specification of the J0 programming language including lexical rules, grammar, type system, and runtime semantics.
tags: [rfc, j0, language-spec, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
rfc_number: "001"
status: Accepted
author: Clinton L. Jeffery
---

# RFC-001 — J0 Language Specification

| Field | Value |
|---|---|
| RFC Number | 001 |
| Status | Accepted |
| Author | Clinton L. Jeffery |
| Created | 2026-06-28 |
| Last updated | 2026-06-28 |

---

## Summary

J0 is a statically-typed, object-oriented subset of Java designed for teaching compiler construction. This RFC is the authoritative specification of J0 syntax and semantics.

---

## 1. Lexical specification

### 1.1 Keywords

```
boolean  break  class  else  false  if  int  new
null     public return static  this  true  void  while
```

### 1.2 Operators and punctuation

| Symbol | Name |
|---|---|
| `+` `-` `*` `/` `%` | Arithmetic operators |
| `==` `!=` `<` `>` `<=` `>=` | Comparison operators |
| `&&` `\|\|` `!` | Logical operators |
| `=` | Assignment |
| `(` `)` `{` `}` `[` `]` | Grouping and indexing |
| `;` `,` `.` | Statement terminator, separator, member access |

### 1.3 Literals

| Token | Pattern | Examples |
|---|---|---|
| `INTLIT` | `[0-9]+` | `0`, `42`, `1000` |
| `BOOLLIT` | `true \| false` | `true`, `false` |
| `STRLIT` | `"([^"\\]|\\.)*"` | `"hello"`, `"line\n"` |
| `NULLLIT` | `null` | `null` |
| `ID` | `[a-zA-Z_][a-zA-Z0-9_]*` | `x`, `myVar`, `Foo` |

### 1.4 Comments

```
// Single-line comment to end of line
/* Multi-line comment — not nested */
```

Whitespace (spaces, tabs, newlines) is ignored except as a token separator.

---

## 2. Grammar (BNF)

```
program       ::= classdecl+

classdecl     ::= 'class' ID '{' memberdecl* '}'

memberdecl    ::= fielddecl
               |  methoddecl

fielddecl     ::= type ID ';'

methoddecl    ::= type ID '(' paramlist? ')' block
               |  'public' 'static' type ID '(' paramlist? ')' block

paramlist     ::= param (',' param)*
param         ::= type ID

type          ::= 'int' | 'boolean' | 'void' | ID | type '[' ']'

block         ::= '{' stmt* '}'

stmt          ::= ';'
               |  block
               |  localvardecl
               |  assignstmt
               |  ifstmt
               |  whilestmt
               |  returnstmt
               |  exprstmt

localvardecl  ::= type ID ('=' expr)? ';'

assignstmt    ::= lvalue '=' expr ';'

lvalue        ::= ID
               |  expr '.' ID
               |  expr '[' expr ']'

ifstmt        ::= 'if' '(' expr ')' stmt ('else' stmt)?

whilestmt     ::= 'while' '(' expr ')' stmt

returnstmt    ::= 'return' expr? ';'

exprstmt      ::= expr ';'

expr          ::= expr '||' expr
               |  expr '&&' expr
               |  expr ('==' | '!=') expr
               |  expr ('<' | '>' | '<=' | '>=') expr
               |  expr ('+' | '-') expr
               |  expr ('*' | '/' | '%') expr
               |  '!' expr
               |  '-' expr
               |  expr '.' ID '(' arglist? ')'
               |  ID '(' arglist? ')'
               |  'new' ID '(' arglist? ')'
               |  'new' type '[' expr ']'
               |  expr '[' expr ']'
               |  expr '.' ID
               |  '(' expr ')'
               |  ID
               |  'this'
               |  INTLIT | BOOLLIT | STRLIT | NULLLIT

arglist       ::= expr (',' expr)*
```

Operator precedence follows Java conventions (highest to lowest):
1. Unary: `!`, unary `-`
2. Multiplicative: `*`, `/`, `%`
3. Additive: `+`, `-`
4. Relational: `<`, `>`, `<=`, `>=`
5. Equality: `==`, `!=`
6. Logical AND: `&&`
7. Logical OR: `||`

---

## 3. Type system

### 3.1 Types

| Type | Description |
|---|---|
| `int` | 32-bit signed integer |
| `boolean` | Boolean: `true` or `false` |
| `void` | Return type of methods that return no value |
| *ClassName* | Reference type; instances of user-defined classes |
| *type*`[]` | One-dimensional array of *type* |
| `null` | The null reference; assignable to any reference type |

### 3.2 Type rules (selected)

- Arithmetic operators (`+`, `-`, `*`, `/`, `%`) require both operands to be `int`; result is `int`.
- Relational operators (`<`, `>`, `<=`, `>=`) require `int` operands; result is `boolean`.
- Equality operators (`==`, `!=`) require both operands to have the same type; result is `boolean`. Reference types are compared by identity.
- Logical operators (`&&`, `||`, `!`) require `boolean` operands; result is `boolean`.
- Array access `a[i]` requires `a` to be an array type and `i` to be `int`; result is the element type.
- Method calls are resolved by class-level method lookup; no overloading.
- `new T()` requires `T` to be a declared class; result is type `T`.
- `new T[n]` requires `n` to be `int`; result is type `T[]`. Elements are zero/null-initialised.
- Assignment `lvalue = expr` requires `expr` type to be compatible with `lvalue` type.

### 3.3 Scope rules

- Class scope: fields and methods are visible throughout the class body.
- Method scope: parameters and local variables are visible from declaration to end of method.
- Block scope: local variables declared in an inner block are not visible outside it.
- No shadowing of class fields by local variables is permitted.

---

## 4. Runtime semantics

### 4.1 Execution model

A J0 program starts by invoking the `main` method of the first class in the source file. The signature must be:

```java
public static void main(String[] args)
```

(J0 does not include `String` as a built-in type; `String` is treated as a pre-declared class with special support for literals and `System.out.println`.)

### 4.2 Built-in support

J0 provides minimal built-in I/O through pre-declared methods:

| Expression | Behaviour |
|---|---|
| `System.out.println(e)` | Prints the string representation of `e` followed by newline |
| `System.out.print(e)` | Prints the string representation of `e` without newline |

### 4.3 Memory model

- Objects are heap-allocated via `new`.
- Arrays are heap-allocated via `new T[n]`.
- Primitive types (`int`, `boolean`) are value types stored on the stack or in object fields.
- J0 does not specify a garbage collection algorithm; the reference implementation uses the JVM GC.

### 4.4 Undefined behaviour

The following are undefined in J0 (implementation-defined):
- Dereferencing a `null` reference (reference implementation throws `NullPointerException`)
- Integer overflow (reference implementation wraps, Java semantics)
- Array out of bounds (reference implementation throws `ArrayIndexOutOfBoundsException`)
- Division by zero (reference implementation throws `ArithmeticException`)

---

## 5. Differences from Java

J0 is a strict *subset* of Java with the following deliberate omissions:
- No generics, wildcards, or type parameters
- No interfaces or abstract classes
- No method overloading
- No static fields or non-static main method
- No exception handling (`try`/`catch`/`throw`)
- No multi-dimensional arrays (only 1D)
- No for-loop (only `while`)
- No string operations beyond `System.out.println`
- No imports (all code in a single source file for v1.0)
- No access modifiers other than `public`

---

## Drawbacks

Keeping J0 as a small subset means some real-world Java programs cannot be compiled with it. This is intentional: the goal is teachability, not compatibility.

## Alternatives considered

- **Use full Java:** Too complex for a single-course compiler.
- **Use a made-up syntax:** Increases student cognitive load; J0's Java-like syntax leverages existing familiarity.
- **Use Python-like syntax:** Would require different lexer patterns; Java subset is more relevant for compiler theory examples.

## Unresolved questions

- Should J0 v2.0 support `for` loops? (Deferred to RFC-005.)
- Should J0 v2.0 support multiple source files? (Deferred to RFC-006.)
