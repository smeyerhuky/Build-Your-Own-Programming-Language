---
type: Index
title: "RFC Index — Conductor Labs"
description: Navigation index for Conductor Labs RFCs. An RFC is a pre-implementation technical architecture proposal that engineers and agents respond to before any TASK files are created or code is written.
resource: /startup/03-rfc/
tags: [rfc, architecture, process, index]
timestamp: 2026-06-28T00:00:00Z
---

# RFC Index

## What Is an RFC?

An RFC (Request for Comments) is a **pre-implementation technical architecture proposal**. Before any code is written or any TASK file is created for a non-trivial system component, an RFC is authored to:

1. Describe the **proposed architecture** in sufficient detail that an engineer or agent can implement it without ambiguity.
2. Surface **alternatives considered** and explain why they were rejected.
3. Identify **open questions** that must be resolved before implementation begins.
4. Establish **success criteria** so that reviewers can verify the implementation matches the proposal.

RFCs are not design docs written after the fact. They are written *before* a single line of production code exists for that component.

## RFC Process

```
Author writes RFC (DRAFT)
        ↓
Stakeholders comment (inline, or via separate notes)
        ↓
Author revises → RFC moves to PROPOSED
        ↓
Core team accepts or requests amendments
        ↓
RFC moves to ACCEPTED
        ↓
TASK files are created referencing the RFC
        ↓
Implementation begins
        ↓
RFC updated to IMPLEMENTED when merged
```

**Key rule:** No TASK file for a system defined in an RFC may be created until that RFC reaches ACCEPTED status. This prevents building against a moving target.

## RFC Status Vocabulary

| Status | Meaning |
|--------|---------|
| `DRAFT` | Being written; not ready for review |
| `PROPOSED` | Ready for stakeholder review |
| `ACCEPTED` | Approved; TASK files may be created |
| `AMENDED` | Accepted with recorded modifications |
| `IMPLEMENTED` | Code is merged; RFC is now historical record |
| `WITHDRAWN` | Rejected or superseded; kept for audit trail |

## RFC Registry

| RFC | Title | Status | Author | Date |
|-----|-------|--------|--------|------|
| [RFC-001](rfc-001-agent-harness.md) | Conductor Agent Harness Architecture | DRAFT | Founder | 2026-06-28 |

## Related Sections

- [PRD](../02-prd/) — product requirements that motivate these RFCs
- [Validation](../04-validation/) — go/no-go gates and experiments that must pass before TASK creation
- [Specs](../05-specs/) — implementation specs that reference accepted RFCs
