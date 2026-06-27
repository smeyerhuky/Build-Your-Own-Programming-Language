---
type: Chapter Section
title: "Ch 8 — Deep dive: tac.java"
description: TAC instruction record and its pseudo-ops.
resource: /ch9/tac.java
tags: [deep-dive, tac, ir]
timestamp: 2026-06-27T00:00:00Z
---

# Deep dive: `tac.java`

```java
class tac {
   String op;
   address op1, op2, op3;

   public void print() {
     switch (op) {
       case "proc":     System.out.println(op + "\t" + op1.region + ",0,0"); break;
       case "end":
       case ".code":
       case ".global":
       case ".string":  System.out.println(op); break;
       case "LAB":      System.out.println("L" + op1.offset + ":"); break;
       default:         System.out.print("\t" + op + "\t" +
                                          op1.str() +
                                          (op2 != null ? "," + op2.str() : "") +
                                          (op3 != null ? "," + op3.str() : ""));
                        System.out.println();
     }
   }
}
```

Note this file lives in `/ch9` in the repo — Ch 8 introduces the
*idea* and the test programs (`arrtst.java`, `clstest.java`,
`funtest.java`); Ch 9 introduces the runnable artifact.

## Pseudo-ops

- `proc <region>,<param-bytes>,<local-bytes>` — open a procedure.
- `end` — close a procedure.
- `.code`, `.global`, `.string` — region markers (assembler-style).
- `LAB L<n>` — declare a label.

## Real ops

Arithmetic (`ADD`, `SUB`, `MUL`, `DIV`, `MOD`, `NEG`), comparisons
(`LT`, `LE`, `GT`, `GE`, `EQ`, `NEQ`), control flow (`GOTO`, `BIF`,
`CALL`, `RETURN`), data movement (`PUSH`, `POP`, `LOCAL`, `LOAD`,
`STORE`). One record per instruction; operands are `address`
descriptors.

## Address regions

```
local   — stack-relative (offset from bp)
global  — code-relative
string  — string-table-relative
imm     — immediate literal
lab     — label number
```

## Cross-references

- [Concept: TAC](/wiki/concepts/three-address-code.md)
- [Concept: IR](/wiki/concepts/intermediate-representation.md)
