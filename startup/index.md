---
type: Startup Bundle
title: "Conductor Labs — Startup Documentation Bundle"
description: Root index for the Conductor Labs OKF documentation bundle. Conductor is a compiled DSL for agentic coding workflows — think Terraform for AI agent pipelines. This bundle covers PR/FAQ, PRD, RFC, validation, specs, design, and business plan.
tags: [conductor-labs, startup, okf, dsl, agent-workflows, compiler, agentic-pdlc]
timestamp: 2026-06-28T00:00:00Z
---

# Conductor Labs — Startup Documentation Bundle

Conductor Labs is building **Conductor**: a compiled domain-specific language for agentic coding workflows. Engineers write declarative `.agent` files that describe multi-step AI pipelines; the Conductor compiler — a full lexer → parser → AST → symbol table → type checker → IR → runtime stack — translates them into real API calls across Anthropic Bedrock, OpenAI, and self-hosted Gemma4 on private GPU. The result is what Terraform is to infrastructure: a portable, auditable, version-controllable source-of-truth for agent pipelines that runs in CI, catches errors at compile time, and never locks you to a single model provider.

The founding context is unusual: the compiler is being built on top of a working compiler-construction course (see [/wiki/index.md](/wiki/index.md)), and the runtime harness is grounded in a rigorous agentic systems architecture document (see [/harness/README.md](/harness/README.md)). Every design decision in this bundle is traceable back to one of those two foundations.

## OKF Type Vocabulary

OKF requires only a `type` field in frontmatter; producers define the vocabulary. This bundle uses:

| Type | Meaning | Example file |
|---|---|---|
| `Startup Bundle` | The bundle root (this file). | `startup/index.md` |
| `Index` | A directory `index.md` whose purpose is navigation. | `startup/01-prfaq/index.md` |
| `Log` | Append-only creation and decision log. | `startup/log.md` |
| `PRFAQ Section` | A press release or internal FAQ page (Amazon working-backwards style). | `startup/01-prfaq/press-release.md` |
| `PRD Overview` | Phase map and top-level success metrics. | `startup/02-prd/index.md` |
| `PRD Phase` | Single phase with agentic task decomposition. | `startup/02-prd/phase-1-mvp.md` |
| `RFC` | Technical architecture proposal with alternatives and trade-offs. | `startup/03-rfc/compiler-ir.md` |
| `Validation` | Go/no-go criteria or designed validation experiments. | `startup/04-validation/index.md` |
| `Spec` | Formal specification (grammar, schema, protocol). | `startup/05-specs/agent-grammar.md` |
| `Design Doc` | Component-level system design with interfaces and data flow. | `startup/06-design/runtime-vm.md` |
| `Business Plan Section` | Any page under `07-business-plan/`. | `startup/07-business-plan/unit-economics.md` |

Consumers tolerate unknown types per the OKF spec.

## Bundle Structure

| Directory | Purpose |
|---|---|
| [`01-prfaq/`](./01-prfaq/index.md) | Amazon-style PR/FAQ — press release and hard internal questions. Start here for product context. |
| [`02-prd/`](./02-prd/index.md) | Product Requirements Document — phase map, task decomposition, success metrics. |
| [`03-rfc/`](./03-rfc/index.md) | RFCs for technical architecture decisions: IR design, GPU tier, MCP integration. |
| [`04-validation/`](./04-validation/index.md) | Validation experiments and go/no-go criteria for each major assumption. |
| [`05-specs/`](./05-specs/index.md) | Formal specifications: `.agent` grammar (EBNF), type system, IR instruction set. |
| [`06-design/`](./06-design/index.md) | Component design docs: compiler pipeline, runtime VM, web IDE, metrics dashboard. |
| [`07-business-plan/`](./07-business-plan/index.md) | Business model, unit economics, funding scenarios, competitive analysis. |

## How to Navigate This Bundle

This bundle is designed for progressive disclosure. Read in this order depending on your goal:

**If you are a new team member or advisor:**
1. Read this file (you are here).
2. Read [`01-prfaq/press-release.md`](./01-prfaq/press-release.md) to understand what we're building and for whom in plain language.
3. Read [`01-prfaq/internal-faq.md`](./01-prfaq/internal-faq.md) to understand the hard questions and honest answers.
4. Read [`02-prd/index.md`](./02-prd/index.md) for the phase map and how we plan to get there.

**If you are an engineer onboarding:**
1. Read [`05-specs/agent-grammar.md`](./05-specs/index.md) for the language spec.
2. Read [`06-design/`](./06-design/index.md) for component interfaces.
3. Cross-reference [`/wiki/index.md`](/wiki/index.md) for compiler theory grounding (the book maps 1:1 to our pipeline).
4. Read [`/harness/README.md`](/harness/README.md) for the runtime architecture North Star.

**If you are evaluating the business:**
1. Read [`01-prfaq/press-release.md`](./01-prfaq/press-release.md).
2. Read [`07-business-plan/index.md`](./07-business-plan/index.md).
3. Read [`04-validation/index.md`](./04-validation/index.md) for our risk map.

**If you are an LLM agent running a build task:**
1. Load `startup/index.md` (this file) first.
2. Load the most specific relevant section (e.g., a single PRD phase or RFC).
3. Cross-link to `/wiki/concepts/` for compiler-theory definitions as needed.
4. Do not load the full bundle at once — it is designed for selective retrieval.

## Related Bundles

| Bundle | Path | Relationship |
|---|---|---|
| Compiler Theory Wiki | [`/wiki/index.md`](/wiki/index.md) | Grounding document. Our compiler pipeline maps chapter-by-chapter to the book (Ch3=Lexer, Ch4=Parser, Ch5=AST, Ch6=Symbol Table, Ch7=Type Checker, Ch8=IR, Ch9/11=Runtime VM). |
| Harness Architecture | [`/harness/README.md`](/harness/README.md) | North Star for the Conductor runtime. The "Most Capable Agent System" architecture describes the agentic OS that Conductor is designed to be infrastructure for. |

## Log

See [`log.md`](./log.md) for the append-only creation and decision log.
