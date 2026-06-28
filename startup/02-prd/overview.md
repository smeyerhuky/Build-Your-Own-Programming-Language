---
type: PRD Overview
title: "Conductor — Product Requirements Overview"
description: Full PRD overview for Conductor, a compiled DSL for agentic coding workflows. Covers product vision, design principles, phase map, success metrics, non-goals, and agentic PDLC methodology.
tags: [prd, overview, conductor, dsl, compiler, agentic-pdlc, bedrock, vllm]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor — Product Requirements Overview

## Product Vision

Conductor is a compiled domain-specific language for declaring, validating, and executing agentic coding workflows. Developers write `.agent` files — declarative pipelines that specify which models to call, what context each step receives, which tools it invokes, and how control flows between steps — and the Conductor compiler translates those files into typed, executable IR that dispatches to LLM APIs (AWS Bedrock, OpenAI, or a private vLLM cluster) and deterministic tools (filesystem, shell, HTTP, human checkpoints). The compiler catches type errors at build time rather than at 2 AM when a production pipeline silently passes `FileList` into a step expecting `ApprovalResult`. Think Terraform for AI agent workflows: the intent is declared, the plan is validated, the execution is auditable, and the state can be checkpointed and resumed. Conductor exists because agentic workflows built in raw Python glue code are brittle, untestable, and impossible to review — every developer reinvents the same retry logic, context-window management, and human-in-the-loop plumbing. Conductor makes that infrastructure a compiler artifact, not a craft skill.

---

## Design Principles

### 1. Portability-First

A `.agent` file should run unchanged on AWS Bedrock Claude 3.5 Sonnet, on a private Gemma4-26B cluster, or on OpenAI GPT-4o. Model selection is a routing decision made at compile or run time, not baked into the pipeline logic. The `.agent` DSL uses named model aliases (`model default: claude-3-5-sonnet`) that resolve at runtime against a tenant config. Swapping providers is a one-line change in `conductor.yaml`, not a rewrite of prompt construction logic. This portability is not cosmetic: it enables the critical path from free tier (Gemma4-12B, fast, cheap) to builder tier (Bedrock, best quality) to enterprise tier (VPC-isolated private GPU) without code changes.

### 2. Compile-Time Correctness

The compiler's type checker knows the return type of every built-in tool: `fs.glob()` returns `FileList`, `shell.run()` returns `ExecResult` (with `.exit_code`, `.stdout`, `.stderr`), `human.checkpoint()` returns `ApprovalResult` (with `.status` of `approved` or `rejected`). If a `branch` node references `.status` on a variable that holds a `FileList`, the compiler emits a `TypeError` with file, line, and column before any API call is made. This is the core value proposition over raw Python: a 20-step pipeline that would fail on step 19 due to a type mismatch fails at compile time, in the editor, with a red squiggle. The cost of compile-time correctness is a type signature registry that must be kept up to date; that cost is worth paying.

### 3. Explicit Context

Every LLM call in a `.agent` pipeline declares exactly what goes into its context window. There are no implicit globals, no hidden state injected by the framework. The `context:` declaration in each step is the complete, auditable list of sources: prior step outputs, file reads, wiki retrievals, and inline literals. The compiler tracks token budgets across the declared context and emits a warning if the assembled context window is projected to exceed the model's limit. This explicitness is not pedantry — it is what makes pipelines reviewable by humans and diffable in pull requests. When a pipeline's behavior changes, the diff shows exactly which context source changed.

### 4. Auditable Pipelines

`.agent` files are plain text, committed to version control, diffable in pull requests, and reviewable by anyone who can read a configuration file. Every execution is logged to Postgres with the full IR, the assembled context per step, the model response, and the tool outputs. Enterprise customers can export the complete audit trail as structured JSON for compliance systems. The combination of version-controlled source and execution-logged runtime means that a pipeline behavior can be traced end-to-end: from the `.agent` source revision, through the compiled IR, to the exact API call that produced a given output. This is the property that makes Conductor suitable for regulated workflows (code generation touching financial logic, security-relevant refactors, etc.).

### 5. Progressive Complexity

A simple two-step pipeline — read files, call a model, write output — should look simple. A pipeline with loops, conditional branches, multi-model routing, human checkpoints, and context budget management should be possible without fighting the language. The `.agent` DSL achieves this by keeping the common path minimal (three keywords: `step`, `prompt`, `output`) and making advanced features additive (`loop:`, `branch:`, `condition:`, `context budget:`). A new user should be able to read the canonical `refactor-dead-code` template and understand it in five minutes. An experienced user should be able to express a 15-step pipeline with retry logic and MCP tool references without hitting a wall. Complexity is unlocked progressively, not front-loaded.

---

## Phase Map

| Phase | Timeline | Primary Goal | Exit Gate |
|---|---|---|---|
| 1 — MVP | Month 1–2 | Core compiler (lexer → IR) + Bedrock runtime + web IDE | 10 paying users ($29/mo Builder tier) |
| 2 — Growth | Month 3–4 | VS Code ext + CLI + MCP + GitHub Actions + team workspaces + 5 templates | 100 MAU, VS Code ext 50+ installs |
| 3 — Scale | Month 5–6+ | vLLM GPU tier + enterprise SSO + AWS Marketplace listing | 3 enterprise LOIs, Marketplace active |

---

## Success Metrics Per Phase

### Phase 1 — MVP

| Metric | Target | Measurement |
|---|---|---|
| Paying users | 10 Builder-tier subscribers | Stripe dashboard |
| Pipeline compilation success rate | >95% on canonical test suite | CI test run |
| End-to-end latency (Bedrock, small pipeline) | <10s compile + dispatch | Run log p95 |
| Compiler self-hosting | Compiler built using Conductor pipelines by Week 6 | Git log |
| Type error detection rate | 100% of 20 negative test cases caught | Test suite |
| Execution state persistence | Resume from checkpoint after forced kill | Integration test |

### Phase 2 — Growth

| Metric | Target | Measurement |
|---|---|---|
| Monthly active users | 100 | Auth/event log |
| VS Code extension installs | 50+ | VS Code Marketplace |
| MCP compliance | All MCP protocol tests passing | MCP test suite |
| Template usage | At least 2 of 5 templates used by >10 distinct users | Usage analytics |
| GitHub Actions integration | Functional in CI on at least 5 public repos | GitHub API |
| Team workspace adoption | At least 3 teams on Team tier ($99/seat/mo) | Stripe |

### Phase 3 — Scale

| Metric | Target | Measurement |
|---|---|---|
| Enterprise LOIs | 3 signed | CRM |
| AWS Marketplace listing | Active, at least 1 PAYG customer | Marketplace console |
| vLLM p50 latency (Gemma4-26B) | <500ms first token | vLLM metrics |
| GPU tier availability | 99.5% over 30 days | Uptime monitor |
| Enterprise SSO | Okta + Azure AD integration tested | QA report |
| VPC deployment | At least 1 customer using VPC Terraform module | Deployment log |

---

## Non-Goals for v1

The following are explicitly out of scope through Phase 1 and Phase 2:

- **Mobile application**: Conductor is a developer tool. No iOS or Android app.
- **No-code / visual pipeline builder**: Drag-and-drop UI is not planned. The `.agent` DSL is the interface. A visual graph view of compiled pipelines may be added as a read-only debug tool, but pipelines are always authored as code.
- **General AI assistant / chat UI**: Conductor is not a Claude or ChatGPT competitor. It is infrastructure for developers building their own agent workflows.
- **Training or fine-tuning our own foundation models**: We run existing models (Gemma4, Claude 3.5 Sonnet) on our infrastructure. We do not train foundation models. Fine-tuning (Phase 3 Beta) is customer-data fine-tuning of Gemma4-12B, not training from scratch.
- **Workflow marketplace with revenue sharing for template authors**: Templates in Phase 2 are free, community-submitted, and curated by the team. A monetized marketplace is not planned before revenue break-even.
- **Execution on local hardware / edge**: All execution runs server-side. Local execution (running `.agent` pipelines against a local Ollama instance) is a community request but not on the roadmap through Phase 3.
- **Support for non-coding workflows**: Conductor's type system, tool registry, and templates are all oriented toward software development tasks. It is not a general-purpose business process automation tool.

---

## Agentic PDLC Methodology

Conductor is built using the methodology it embodies: **Agentic PDLC** (Product Development Lifecycle). Tasks in the phase documents are not written for human engineers who will spend days on each one. They are written for LLM coding agents — specifically, for Claude Code running against the same harness that Conductor is building.

This has concrete implications for how tasks are sized and specified:

**Task sizing follows agent session limits.** A coding agent session has a practical ceiling — context window, compute budget, attention drift. Tasks labeled S (under 4 hours) are self-contained enough that a fresh agent can complete them without carrying context from a previous session. Tasks labeled M (4–16 hours) require the agent to hold a moderate amount of prior context — usually from one or two dependency tasks — but do not span multiple distinct problem domains. Tasks labeled L (16–40 hours) are larger integration tasks that a single agent can drive in a long session with a detailed specification.

**Tasks include explicit agent context.** Every task lists the files, wiki pages, and prior task outputs the agent needs before starting. An agent that starts a task without reading the specified context is likely to produce subtly wrong output. The context list is not aspirational — it is the minimum viable reading list.

**Acceptance criteria are testable by the agent.** Each criterion is phrased so that the agent can verify it programmatically before marking the task done: `npm test passes`, `compiler emits TypeError on line 7 of negative-test-04.agent`, `POST /compile returns HTTP 200 with IR`. Criteria that say "the code should be clean" or "the UX should feel good" are not in this document.

**The founder reviews, not rebuilds.** The founder's role at each task boundary is to read the agent's diff, run the acceptance criteria, and merge or send back with a note. The founder is not expected to rewrite agent output — if an agent's output requires a full rewrite, the task was under-specified and the task definition should be improved.

**The compiler is self-hosting by Phase 1 Week 6.** The Conductor compiler is itself built using Conductor pipelines — once the IR emitter and Bedrock dispatcher are functional (TASK-106, TASK-107), the harness can run `.agent` pipelines that perform compiler development tasks. This is both a milestone and a stress test: if Conductor cannot be used to build Conductor, something is wrong with the expressiveness of the DSL or the reliability of the runtime.

For the underlying reliability math — why step-level error rates compound destructively in long pipelines, why deterministic rails must be used for mandatory steps, and what the capability acquisition ladder looks like — see `harness/README.md`. The Agentic PDLC is an application of those principles to the software development lifecycle itself.
