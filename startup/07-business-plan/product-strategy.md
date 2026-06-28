---
type: Business Plan Section
title: "Conductor Labs — Product Strategy"
description: Product strategy and positioning for Conductor. Covers positioning statement, what Conductor is not, three structural moats, open-core vs closed-source decision rationale, and phased product roadmap through enterprise.
tags: [product-strategy, positioning, roadmap, conductor, open-core]
timestamp: 2026-06-28T00:00:00Z
---

# Product Strategy

## Positioning Statement

**"Conductor is the programming language for AI agent workflows. Write a `.agent` file, compile it, run it anywhere. When it breaks, you get a type error — not a 2 AM Slack alert."**

This positioning is precise by design. It says:
- What it is (a programming language — not a library, not a visual builder, not a chat interface)
- What you do with it (write a file, compile it, run it)
- What the key differentiator is (type errors at compile time, not runtime surprises)

Every marketing message, every HN post, every demo should return to this. The positioning is for professional developers. It should sound completely alien to a non-developer — that is intentional.

---

## What We Are NOT

Clarity about what Conductor is NOT is as important as clarity about what it is. The biggest risk is being positioned in the wrong category — either by confused press coverage, inaccurate user descriptions, or founder wavering under investor pressure to "expand the market."

**Not a no-code builder** (that's Lovable, n8n, Dify — different abstraction level, different buyer, different pricing ceiling). A developer who wants to avoid writing code is not our customer. A developer who wants to write better, more reliable code is.

**Not an IDE copilot** (that's Cursor, GitHub Copilot — code assistance during authoring, not pipeline infrastructure). Cursor helps you write Python. Conductor replaces the Python you would have written with a compiled language.

**Not a chat interface** (that's Claude.ai, ChatGPT — conversational AI, not developer infrastructure). Conductor is not something you talk to; it is something you compile and run.

**Not a Python library** (that's LangChain, LangGraph — add-on packages within Python). Conductor is a language with its own syntax, type system, and compiler. The distinction matters: you cannot add a type checker to LangChain as a library without rewriting LangChain. A language compiles; a library executes.

**Not a managed platform** (that's Claude Projects, AWS Bedrock Agents — provider-managed, provider-locked). Our value is precisely that we are neutral and self-hostable.

---

## The Three Moats

### Moat 1: Provider Neutrality

**No LLM provider will ever build a product that routes to its competitors.**

This is the structural insight that defines Conductor's defensibility. Anthropic's commercial incentive is to keep developers on Anthropic models. OpenAI's commercial incentive is to keep developers on OpenAI models. Neither will build a product whose primary feature is routing away from them.

We can — and that is the point.

Provider neutrality compounds over time:
- When a new provider launches (Grok, Gemini, future open-weights), Conductor adds a backend; existing `.agent` files automatically have access to the new model
- When a provider has an outage, Conductor pipelines can failover to an alternative in one config change
- When pricing changes dramatically (as it has with each generation), Conductor users can arbitrage models by switching a single line

The neutrality moat is structural and permanent. It does not require Conductor to be technically superior to Anthropic's own tooling — it requires only that Anthropic cannot and will not build the neutral alternative.

### Moat 2: Ecosystem Network Effects

`.agent` files, MCP adapters, community templates, GitHub Actions integrations — each artifact accretes to the platform rather than any individual user.

The compounding mechanism:
1. Developer writes a `.agent` pipeline → checks it into their repo
2. Another developer on the team reads it, modifies it, extends it — now there are two Conductor users
3. Developer publishes a blog post about their pipeline → 50 people try the example
4. Developer submits the pipeline to the community template library → 500 more users discover it

The template marketplace (Phase 3) formalizes this: community members earn revenue for published templates (70/30 split). This creates financial incentive to create Conductor content — the ecosystem generates its own growth fuel.

GitHub Actions integration is the CI/CD flywheel: when `.agent` files run in a company's CI pipeline, every engineer who touches that CI pipeline sees a Conductor file. The company-internal discovery surface compounds independently of our marketing.

MCP compatibility is the tool ecosystem flywheel: every MCP tool vendor (there are hundreds now) is a potential Conductor tool, without any integration work on our part. We inherit the MCP ecosystem for free.

### Moat 3: Compiler Expertise

Building a correct type-checker and IR for agent workflows requires programming language theory knowledge that is genuinely rare in the agent tooling ecosystem. Most "orchestration frameworks" are built by ML engineers who are experts in models, not type systems.

The Conductor type system needs to handle:
- Algebraic data types for tool return values (`FileList | ExecResult | ErrorResult`)
- Context window budget tracking as a first-class type constraint
- Prompt template type-checking (variable substitution into typed slots)
- Branch condition type narrowing (after `on: approved`, the downstream step knows `$checkpoint_result.status == "approved"`)
- MCP tool schema validation at compile time (tool contracts defined by MCP schemas become type constraints)

This is not library glue. This is programming language design. The foundation — documented in `wiki/` and implemented in the compiler pipeline — represents the technical credibility moat that "a team that built a Python library" cannot replicate in a quarter.

---

## Open-Core vs Closed-Source Decision

**Decision: Open-core. Compiler and runtime are Apache 2.0. Cloud hosting, team features, GPU tier, and enterprise controls are closed-source SaaS.**

This follows the n8n model. Rationale:

**Why open-source the core?**
1. GitHub stars drive PLG adoption — developers trust products they can read and fork
2. "Open-core prevents fear of lock-in" — enterprise buyers are more willing to adopt infrastructure with an open-source escape hatch
3. Community contributions improve the compiler and add MCP adapters — free engineering work from aligned contributors
4. Technical credibility — a public compiler that PL-theory literate developers can read signals that the implementation is serious

**Why close the cloud/team/enterprise layer?**
1. Revenue moat — the subscription value is in orchestration, team management, GPU serving, and compliance features that depend on hosted infrastructure
2. Prevents resale — AWS, Azure, and Google cannot package Conductor into their managed services without a partnership (cf. n8n's fair-code approach to this problem)
3. Enterprise features require operational expertise — SSO, VPC deployment, audit logs, and SLA are not things a self-hoster can easily replicate

**License choice**: Consider the Business Source License (BSL, used by HashiCorp/Terraform) or SSPL (used by MongoDB) for the core if we need to prevent cloud-provider resale. Apache 2.0 is simpler and more trusted; decide at launch based on community feedback. The n8n "Sustainable Use License" (fair-code) is a pragmatic middle path worth evaluating.

---

## Product Roadmap

### Phase 1 (Month 1-2): MVP — Core Language + Bedrock

**Goal**: 10 paying users with real pipelines running.

Deliverables:
- `.agent` language spec v0.1 (lexer, parser, AST, type checker, IR)
- Runtime: Bedrock Claude 3.5 Sonnet + OpenAI GPT-4o
- Web IDE: syntax highlighting, compile, run — no local install required
- 5 built-in tools: `fs.glob`, `fs.read`, `fs.write`, `human.checkpoint`, `exec.shell`
- MCP tool adapter (any MCP server is immediately available as a tool)
- CLI: `conductor compile`, `conductor run`, `conductor check` (type-check only)
- `pip install conductor-lang` working

Success criteria: 10 beta users with at least one real pipeline running. 3 paying Builder subscribers.

### Phase 2 (Month 3-4): Distribution Unlocks — VS Code + MCP + CI

**Goal**: HN Show HN launch drives 100-500 signups.

Deliverables:
- VS Code extension: syntax highlighting, Intellisense, inline type errors, run-from-editor
- GitHub Actions integration: `conductor/run-agent@v1` action (`.agent` files in CI)
- Community template library (5 launch templates: code review, documentation generation, dead code removal, test writing, PR summarization)
- Metrics dashboard (Builder tier): execution count, token usage, step latency, error rate
- `conductor init` command: generates `.agent` scaffold from description

Post-MVP priority note: The VS Code extension is the highest-leverage distribution unlock. Even if it "feels like Phase 2," ship it before the HN launch. Developers judge tools by their editor integration, not their web IDE.

Success criteria: 500 GitHub stars, 25 Builder subscribers, first Team subscription.

### Phase 3 (Month 5-6): GPU Tier + Team Workspace + AWS Marketplace

**Goal**: Private GPU tier live. Team workspaces. AWS Marketplace listing.

Deliverables:
- Private GPU tier: Gemma4-26B-A4B + MTP on A10G spot, deployed in 3 regions
- Gemma4-12B + Q4 for Free tier (replaces Tier 3)
- Team workspace: shared pipeline library, team member management, shared secrets vault
- AWS Marketplace listing + ISV Accelerate qualification
- SOC2 Type I (engage auditor in Month 4, certify by Month 6)
- Usage-based pricing for GPU tier: credit packs at $10/10k, $80/100k, $600/1M

Success criteria: GPU tier live and measurably faster/cheaper than Bedrock for target workloads. 5 Team subscribers. AWS Marketplace listing active.

### Phase 4 (Month 7+): Enterprise + Ecosystem

**Goal**: First $2k+/mo enterprise customer. Expand ecosystem integrations.

Deliverables:
- SSO/SAML integration (Okta, Azure AD, Google Workspace)
- VPC deployment option (bring-your-own-AWS account)
- Audit logs + compliance export (SOC2 Type II preparation)
- LangGraph export target: `conductor export --target=langgraph` generates Python LangGraph code from `.agent` file
- Fine-tuning hooks: annotate pipeline outputs for LoRA training on Gemma4
- Anthropic partnership exploration: volume pricing on Bedrock in exchange for co-marketing

Success criteria: 1 enterprise contract signed at $2k+/mo. 80 Builder subscribers. 30 Team seats.

---

## Technical Architecture Philosophy

**Conductor is not a wrapper around LLM APIs.** It is a compiler that targets LLM APIs.

The distinction matters for engineering decisions:
- Language design is permanent (breaking changes require major version bumps — don't be cavalier)
- The type system must be sound (a type error should always be a real error — no false positives that erode trust)
- The IR is the portability layer — adding a new runtime target (LangGraph, DAGster, etc.) should require adding a new IR backend, not changing the compiler frontend
- Error messages are a product feature — a bad type error message is a bug, not a "could be improved"

Every compiler decision should be made asking: "Would a TypeScript developer or a Terraform user find this behavior intuitive?" Those are our analogies, not "would a LangChain user find this familiar."
