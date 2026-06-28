---
type: Index
title: "Conductor Design Documentation"
description: Navigation index for Conductor system design documents, covering architecture, compiler, runtime, GPU serving, billing, and data model. Anchored by the Most Capable Agent System as the North Star for runtime reliability philosophy.
tags: [conductor, design, architecture, index]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor — Design Documentation

This section contains the technical design documents for Conductor, a compiled DSL for agentic coding workflows. Each document covers a distinct system layer in depth, with ASCII diagrams, schema tables, and real implementation guidance.

## North Star

The runtime design philosophy is anchored in **[harness/README.md](../../harness/README.md)** — "Most Capable Agent System." Key principles drawn from that document and applied throughout these designs:

- Reliability compounds across steps: design for the march of nines, not for demos
- If something must happen every time, codify it in the harness (not a prompt)
- Specialized harnesses over general-purpose ones for high-value repeated workflows
- Checkpoint and resume as first-class primitives, not afterthoughts
- Every phase leaves a file or artifact trail; run state must be queryable from the control plane
- Idempotency keys on all side-effecting operations
- Human-in-the-loop at meaningful decision points only

---

## Documents

| File | Type | Description |
|---|---|---|
| [system-architecture.md](./system-architecture.md) | Design Doc | Full system architecture: component diagram, data flow from compile to run, external integrations, scalability, security, and disaster recovery. |
| [compiler-design.md](./compiler-design.md) | Design Doc | Multi-phase compiler pipeline: lexer, parser, AST, task graph, symbol table, type checker, and IR emitter. Maps directly to chapters 3–8 of the build-your-own-programming-language curriculum. |
| [runtime-design.md](./runtime-design.md) | Design Doc | IR walker loop, dispatch table, model router, execution state persistence, reliability design, context stack, and SSE event stream. Implements harness/README.md reliability principles. |
| [gpu-serving-design.md](./gpu-serving-design.md) | Design Doc | vLLM serving layer for Gemma4 private and fast tiers: MTP speculative decoding, multi-region deployment, spot interruption handling, and cost model. |
| [billing-design.md](./billing-design.md) | Design Doc | Three-component billing: Stripe subscription, credit system, and AWS Bedrock pass-through reconciliation. Includes data model, budget enforcement, and dashboard data. |
| [data-model.md](./data-model.md) | Design Doc | Complete Postgres schema with row-level security: DDL for all tables, ER diagram, RLS policies, key indexes, and migration strategy. |

---

## Component Map

```
startup/06-design/
├── index.md                  ← this file
├── system-architecture.md    ← how all pieces fit together
├── compiler-design.md        ← .agent source → IR JSON
├── runtime-design.md         ← IR JSON → execution
├── gpu-serving-design.md     ← Tier 2/3 model serving
├── billing-design.md         ← credits, Stripe, Bedrock reconciliation
└── data-model.md             ← Postgres schema (control plane DB)
```

## Reading Order

For new engineers: start with **system-architecture.md** to build the mental model, then read **compiler-design.md** and **runtime-design.md** as the core product loop. Read **gpu-serving-design.md** before working on the model router, **billing-design.md** before touching the payment path, and **data-model.md** as reference when writing migrations or queries.
