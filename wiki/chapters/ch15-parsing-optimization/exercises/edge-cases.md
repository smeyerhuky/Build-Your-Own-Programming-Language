---
type: Edge Case Catalog
title: "Ch 15 — Edge cases (GC anti-patterns)"
description: Common GC pitfalls.
tags: [edge-cases, cot, gc]
timestamp: 2026-06-27T00:00:00Z
---

# Edge cases — Ch 15 garbage collection

## 1. Refcount cycle leak

### Bad input
```
obj1.ref = obj2
obj2.ref = obj1
clear all roots
```

### Symptom
Memory usage never drops — refcounts stay at 1 because each object
references the other.

### CoT trace
1. Refcounting can't break cycles by itself.
2. Either add a cycle collector (tri-color trial deletion) or pair
   it with a backup mark-sweep.
3. Or document that cyclic references must use weak refs.

### Lesson
Refcounting's appeal is determinism, but the cycle problem is a
hard ceiling on what it can collect.

---

## 2. Conservative scan misidentifying ints as pointers

### Bad input
A stack slot holds `0x7fff_ffff` (a large int). The collector treats
it as a pointer into the heap.

### Symptom
A live heap object whose address happens to match is kept alive
spuriously; in pathological cases, the collector reads "through" it
and crashes.

### CoT trace
1. Conservative collectors don't know which slots are pointers.
2. They scan everything that *could* be a pointer.
3. Trade-off: simplicity vs. false retention.

### Fix
Either use a precise GC (compiler emits a stack map per call site)
or accept the over-approximation and design heaps so non-pointer
values can't alias real addresses.

### Lesson
The collector's view of "what is a pointer" is part of the runtime
contract with the compiler. Decide it explicitly.

---

## 3. Forgetting GC-safe points

### Bad input
A long-running native helper holds a heap pointer in a register
during a collection.

### Symptom
After GC moves the object (compacting collector), the register
still holds the old address; the next dereference crashes.

### CoT trace
1. Compacting GC requires the collector to update every live
   pointer.
2. Pointers the collector can't see (in registers, in unboxed
   structs) can't be updated.
3. Either use safe-points (only collect when registers are spilled)
   or pin objects that must survive register liveness.

### Lesson
Compaction makes locality better but adds a contract every backend
must obey. Pick a non-moving collector unless you're ready to enforce
safe-points.
