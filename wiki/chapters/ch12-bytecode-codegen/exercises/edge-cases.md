---
type: Edge Case Catalog
title: "Ch 12 — Edge cases"
description: Common bytecode-emission bugs.
resource: /ch12/byc.java
tags: [edge-cases, cot, bytecode-codegen]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 12 bytecode codegen

## 1. Duplicate strings without intern

### Buggy input
A new emitter writes each string literal to the string table
directly:

```java
int off = stringtab.append(s); // always appends
```

### Symptom
Programs that print the same literal twice carry two copies in the
`.j0` file. Comparing strings by address (`a == b`) breaks.

### CoT trace
1. The string table is supposed to dedup so the same content has the
   same offset.
2. Without dedup, identity comparison fails for distinct-but-equal
   literals.
3. Switch to `intern(s)` which checks an existing-entry map first.

### Fix
```java
int off = stringtab.intern(s);
```

### Lesson
String interning is the cheapest correctness win in any bytecode
emitter — and breaking it is hard to spot because *functional*
output looks fine.

---

## 2. Label collision across procedures

### Buggy input
The label counter is reset to 0 inside `gencode("MethodDecl")`.

### Symptom
Two methods both have labels `L1`, `L2`, … and `CALL`/`GOTO` to
wrong targets after back-patching merges the label maps.

### CoT trace
1. Labels are file-global once they hit the back-patch table.
2. Resetting the counter per procedure makes two distinct labels
   share an id.
3. Use a module-level `serial` for labels too.

### Fix
Allocate label ids via `serial.next()` (or a dedicated `labelSerial`)
that's never reset.

### Lesson
Names that escape a scope must be allocated from a single
namespace. The temptation to "reset for cleanliness" is the bug.

---

## 3. Off-by-eight on stack offset

### Buggy input
The first local is placed at offset `0` (relative to `bp`).

### Symptom
`POP` at `R_STACK 0` overwrites the saved `bp` or return address,
corrupting the call chain.

### CoT trace
1. Frame layout: `[ret_ip | saved_bp | local 1 | local 2 | ...]`.
2. The first local is at offset `+8` (below the saved `bp`) for J0's
   downward-growing stack, but the canonical convention is `-8`
   from `bp` for the first local (locals area starts below the
   saved bp).
3. Refer to `j0machine.java`'s `CALL`/`RETURN` for the exact
   convention before laying out locals.

### Fix
Start locals at `-8` (or whatever `j0machine` documents), not `0`.

### Lesson
Frame layout is a convention enforced by *both* the codegen and the
VM. Picking the layout in only one place silently breaks the other.

---

## 4. Forward jump never patched

### Buggy input
The compiler emits `BIF Lend` with the operand left at `0` and
forgets to record the site for back-patching.

### Symptom
At runtime, `BIF` succeeds and jumps to byte `0` — which is inside
the magic header. The VM either hangs or hits "bad opcode".

### CoT trace
1. Forward jumps must be back-patched when their target's offset is
   known.
2. Forgetting to record the site means the placeholder `0` operand
   is the final value.
3. Add the site to the back-patch list at emit time.

### Fix
```java
int site = icode.size();
icode.add(new tac("BIF", new address("lab", labelId)));
backpatch.computeIfAbsent(labelId, k -> new ArrayList<>()).add(site);
```

### Lesson
Forward-jump back-patching has two halves: record at emit, resolve
at label declaration. Code review should verify both halves are in
the same commit.
