---
type: RFC Index
title: "LangCraft RFCs — Master Index"
description: Master index of all LangCraft Request for Comments (RFC) specifications with status, owner, and summary.
tags: [rfc, index, spec, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# RFC Index

All LangCraft technical specifications are maintained as numbered RFCs. See [`process.md`](./process.md) for the RFC lifecycle and how to propose a new RFC.

---

## Active RFCs

| RFC | Title | Status | Owner | Updated |
|---|---|---|---|---|
| [RFC-001](./rfc-001-j0-language-spec.md) | J0 Language Specification | **Accepted** | Clinton L. Jeffery | 2026-06-28 |
| [RFC-002](./rfc-002-bytecode-format.md) | J0 Bytecode Format | **Accepted** | Clinton L. Jeffery | 2026-06-28 |
| [RFC-003](./rfc-003-stdlib-design.md) | J0 Standard Library Design | **Draft** | Clinton L. Jeffery | 2026-06-28 |
| [RFC-004](./rfc-004-toolchain-integration.md) | J0 Toolchain IDE/LSP Integration | **Draft** | Clinton L. Jeffery | 2026-06-28 |

---

## Status definitions

| Status | Meaning |
|---|---|
| `Draft` | Author is writing; not yet ready for community review |
| `Review` | Open for comments; review deadline in the document |
| `Accepted` | Approved and binding; implementation may proceed |
| `Rejected` | Formally declined; reason documented in the RFC |
| `Withdrawn` | Author withdrew the proposal |
| `Superseded` | Replaced by a newer RFC (link in the document) |

---

## How to propose a new RFC

See [`process.md`](./process.md). In short:
1. Copy the [RFC template](./process.md#template) into a new file `rfc-NNN-short-title.md`.
2. Assign the next available number.
3. Set status to `Draft`.
4. Open a pull request; tag it `rfc`.
5. After a 2-week review period, the CTO accepts or rejects via PR comment.
