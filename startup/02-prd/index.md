---
type: Index
title: "PRD Index — Conductor"
description: Navigation index for all Conductor Product Requirements Documents. Links to the PRD overview and three phase documents covering MVP through enterprise scale.
tags: [prd, index, conductor, agentic-pdlc]
timestamp: 2026-06-28T00:00:00Z
---

# Product Requirements Documents — Conductor

## What a PRD Is For in Agentic PDLC

In traditional software, PRDs are written for human engineers. In Conductor's **Agentic PDLC** methodology, PRDs serve a different purpose: they are the primary specification artifact consumed by LLM coding agents executing each task. Every task in these documents is sized, scoped, and contextually grounded so that a single coding agent session can take the task from specification to merged, tested code without human hand-holding.

This means PRDs here are not aspirational prose. They are operational contracts. Each task includes:

- An explicit complexity rating (S/M/L) calibrated to coding agent session lengths
- A precise list of agent context (which files the agent must read before starting)
- Testable acceptance criteria the agent can verify before closing the task
- A named deliverable — a concrete artifact checked into the repo

The founder's role shifts from writing code to reviewing agent output, resolving ambiguity at task boundaries, and merging approved work. The compiler itself is built by agents running on the emerging harness — Conductor is genuinely self-hosting by Phase 1 Week 6.

All tasks in these documents are sized for LLM coding agent execution: **S = under 4 hours, M = 4–16 hours, L = 16–40 hours** of agent session time.

---

## Documents in This Section

### [overview.md](./overview.md)
**Type**: PRD Overview

Product vision, design principles, phase map, cross-phase success metrics, non-goals for v1, and the Agentic PDLC methodology explanation. Read this first to understand what Conductor is and why it is being built this way.

### [phase-1-mvp.md](./phase-1-mvp.md)
**Type**: PRD Phase

15 tasks covering the core compiler pipeline (lexer through IR), AWS Bedrock runtime dispatcher, execution state machine, Postgres schema, Stripe billing, FastAPI backend, and minimal web IDE. Target: 10 paying Builder-tier users running real agent pipelines.

### [phase-2-growth.md](./phase-2-growth.md)
**Type**: PRD Phase

10 tasks covering VS Code extension, conductor CLI, GitHub Actions integration, MCP server and client, pipeline template marketplace, metrics dashboard, team workspaces, OpenAI provider, and LangChain migration tooling. Target: 100 monthly active users.

### [phase-3-scale.md](./phase-3-scale.md)
**Type**: PRD Phase

10 tasks covering vLLM GPU infrastructure (Gemma4 + MTP speculative decoding), multi-region load balancing, enterprise SSO/SAML, VPC deployment, AWS Marketplace listing, pipeline governance, LangGraph export, fine-tuning service, and SLA monitoring. Target: first enterprise deal, AWS Marketplace live.

---

## Phase Timeline Summary

| Phase | Timeline | Primary Deliverable | Success Gate |
|---|---|---|---|
| 1 — MVP | Month 1–2 | Core compiler + Bedrock runtime + web IDE | 10 users paying $29/mo (Builder tier) |
| 2 — Growth | Month 3–4 | VS Code ext + CLI + MCP + team features + templates | 100 MAU, VS Code ext 50+ installs |
| 3 — Scale | Month 5–6+ | GPU serving + enterprise SSO + AWS Marketplace | 3 enterprise LOIs, Marketplace listing active |

---

## Notes for Agents Reading This Index

- Cross-reference `harness/README.md` for the reliability math and capability acquisition ladder that governs how the harness itself is built.
- Cross-reference `wiki/` for compiler theory (lexer through IR) that underpins tasks TASK-101 through TASK-106.
- All task IDs are stable and used as dependency references across documents. Never renumber a shipped task.
- The startup directory follows OKF frontmatter conventions. Every file begins with the YAML block above.
