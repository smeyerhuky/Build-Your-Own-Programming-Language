---
type: RFC Process
title: "LangCraft RFC Process"
description: The LangCraft RFC lifecycle — how technical specifications are proposed, reviewed, accepted, and maintained.
tags: [rfc, process, governance, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# RFC Process

LangCraft uses a lightweight RFC (Request for Comments) process to make technical decisions in a durable, reviewable, and transparent way. Every significant change to the J0 language, bytecode format, standard library, or platform architecture must be described in an RFC before implementation begins.

---

## Scope — what requires an RFC

An RFC is **required** for:
- Changes to J0 language syntax or semantics
- Changes to the `.j0` bytecode format (opcodes, file structure)
- New standard library modules
- Breaking changes to the compiler API or CLI interface
- New platform architecture decisions (database schema, API shape, auth flow)

An RFC is **not required** for:
- Bug fixes that don't change observable behaviour
- Documentation updates
- Test additions
- Refactoring without behaviour change
- Tooling or CI configuration

---

## Lifecycle

```
Proposed → Draft → Review → [Accepted | Rejected | Withdrawn]
                                  │
                              Superseded (if a newer RFC replaces it)
```

### Stage: Draft
- Author creates a file `rfc-NNN-short-title.md` from the template below.
- Sets `status: Draft`.
- Commits to a branch and opens a PR tagged `rfc`.
- The draft may be iterated on freely.

### Stage: Review
- Author sets `status: Review` and announces in the team discussion channel.
- **Review period: 14 calendar days** minimum.
- Anyone may comment on the PR.
- Author must respond to all substantive comments.

### Stage: Accepted
- The CTO (Clinton L. Jeffery) approves the PR, sets `status: Accepted`, and merges.
- From this point, the RFC is binding. Implementation must conform to it.
- If implementation reveals an error, open a new RFC to amend or supersede.

### Stage: Rejected
- The CTO closes the PR with a written explanation, sets `status: Rejected`.
- The file is merged as-is (with `Rejected` status) for historical record.

### Stage: Withdrawn
- Author closes their own PR; sets `status: Withdrawn`.

### Stage: Superseded
- A new RFC is accepted that replaces this one.
- The old RFC's status is set to `Superseded` with a link to the new RFC.

---

## SLAs

| Stage transition | Target |
|---|---|
| Draft → Review | Author's discretion |
| Review open → CTO decision | 14 days after review period ends |
| Accepted → Reference implementation | 60 days (unless stated otherwise in the RFC) |

---

## Template

Copy this block into `rfc/rfc-NNN-short-title.md` to start a new RFC:

```markdown
---
type: RFC
title: "RFC-NNN — Short Title"
description: One sentence describing what this RFC specifies.
tags: [rfc, langcraft, startup-business]
timestamp: YYYY-MM-DDT00:00:00Z
rfc_number: NNN
status: Draft
author: Your Name
---

# RFC-NNN — Short Title

## Summary

One paragraph explaining what this RFC proposes and why.

## Motivation

What problem does this solve? Why now?

## Detailed design

The full specification. Use tables, grammars, and code blocks as needed.

## Drawbacks

What are the downsides or risks of this approach?

## Alternatives considered

What other approaches were considered? Why was this one chosen?

## Unresolved questions

What is not yet decided? Tag with the reviewer's name if assigned.
```

---

## Numbering

RFCs are numbered sequentially starting at 001. Numbers are never reused. See the [RFC Index](./index.md) for the current highest number.
