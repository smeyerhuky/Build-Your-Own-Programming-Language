---
type: Cookbook
title: "Ch 11 — Cookbook"
description: Canonical patterns for .j0 file handling.
resource: /ch11/j0machine.java
tags: [cookbook, bytecode-format]
timestamp: 2026-06-27T00:00:00Z
---

# Cookbook — Ch 11 file format

## 1. Bump the magic version

### Prompt
"Introduce a v2 bytecode format. Keep v1 loadable."

### Reference solution
1. Change `magstr` to `"Jzero!2\0"`.
2. In `loadbytecode`, search for **both** magics; if v1 is found,
   run a compatibility shim.

```java
int v2 = find("Jzero!2\0".getBytes(...), code);
int v1 = (v2 < 0) ? find("Jzero!!\0".getBytes(...), code) : -1;
if (v2 >= 0) { /* v2 path */ }
else if (v1 >= 0) { upgradeV1ToV2(code); /* fall through */ }
else return false;
```

### Why this is canonical
Keep both magics in the loader; never re-use the old magic for new
semantics.

---

## 2. Add a self-execution shebang

### Prompt
"Let me run `./hello.j0` directly."

### Reference solution
Prepend `#!/usr/bin/env j0x\n` to the file *before* the magic. The
loader's `find()` skips past it automatically.

### Why this is canonical
The whole reason `find()` is a search, not a fixed check.

---

## 3. Encode a 48-bit operand

### Prompt
"Write the 48-bit operand for a `BIF Lend` instruction where Lend
is at byte offset 800."

### Reference solution
```java
// op=BIF, opr=R_ABS, operand=800
out.write(Op.BIF);
out.write(Op.R_ABS);
long v = 800;
for (int j = 0; j < 6; j++) { out.write((int)(v & 0xff)); v >>= 8; }
```

### Why this is canonical
Little-endian, 6 bytes. Matches `getOpnd`'s decoding loop.

---

## 4. Reserve a noop slot for future operands

### Prompt
"Add a 64-bit operand-extension byte without changing the magic."

### Reference solution
Repurpose the `opr` register-mode byte: define a new mode
`R_OPR_EXT = 5` whose meaning is "operand is in the **next**
instruction's bytes 2–7." Decoder: when `opr == R_OPR_EXT`, do
`fetch()` twice and concatenate.

### Why this is canonical
The format has a single free byte (`opr`). Repurposing it without
changing instruction size keeps backward compatibility.
