---
type: Index
title: "Validation Index — Conductor Labs"
description: Navigation index for Conductor Labs validation documents. Validation covers pre-build experiments (2-week discovery sprint) and per-phase go/no-go gates that must pass before advancing to the next build phase.
resource: /startup/04-validation/
tags: [validation, experiments, go-no-go, metrics, index]
timestamp: 2026-06-28T00:00:00Z
---

# Validation Index

## Purpose

Before writing a line of production code, we validate the key assumptions that the business and architecture rest on. This section contains two categories of validation documents:

1. **Experiments** — pre-build, time-boxed tests (typically 1–2 weeks each) that validate or invalidate a specific hypothesis. Experiments run *before* Phase 1 code is written. They are cheap to run and expensive to skip.
2. **Go/No-Go Criteria** — per-phase gates with numeric, measurable thresholds. At the end of each phase, we evaluate the criteria and decide whether to advance, pivot, or shut down. There is no "maybe" — every criterion has a binary pass/fail.

## Why Validate First?

The most expensive mistake in a startup is building the wrong thing at high quality. Validation is the practice of finding out whether we're building the wrong thing before we invest 2 months of engineering time. The pre-build experiments in this section cost approximately 40 hours of founder time and $200 in infrastructure. Phase 1 engineering costs approximately 400 hours. Running the experiments first has a 20× leverage ratio.

## Documents

| Document | Type | Contents |
|----------|------|----------|
| [go-no-go-criteria.md](go-no-go-criteria.md) | Validation | Numeric go/no-go gates for every phase plus kill criteria and a technical risk register |
| [experiments.md](experiments.md) | Validation | Five pre-build experiments with hypotheses, methods, success criteria, timelines, and decision rules |

## Related Sections

- [PRD](../02-prd/) — product requirements that define what we're validating against
- [RFC-001](../03-rfc/rfc-001-agent-harness.md) — architecture proposal whose assumptions these experiments test
- [Specs](../05-specs/) — implementation specs created only after go/no-go gates pass
