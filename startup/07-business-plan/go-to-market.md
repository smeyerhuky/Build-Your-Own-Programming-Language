---
type: Business Plan Section
title: "Conductor Labs — Go-To-Market Strategy"
description: PLG-first GTM for Conductor Labs from zero to $1M ARR. Covers the no-outbound philosophy, ranked channel ROI table, MCP/AWS/Anthropic partnership sequencing, and a day-by-day Launch Week playbook for HN and GitHub.
tags: [go-to-market, gtm, plg, launch, conductor, partnerships]
timestamp: 2026-06-28T00:00:00Z
---

# Go-To-Market Strategy — Conductor Labs

## Philosophy: PLG Only Until $1M ARR

No outbound. No SDRs. No cold email. No conference booths.

Every dollar and every hour before $1M ARR goes into making the product so obviously good that developers tell each other about it unprompted. The YC data is unambiguous: the best dev tool companies (Cursor, Tailwind, n8n, Supabase) all crossed $1M ARR entirely through PLG before adding any sales motion.

Outbound sales before product-market fit is a tax on the founder's time. It produces revenue that feels real but masks that you haven't solved distribution. PLG revenue is sticky and scalable. Outbound revenue is rented.

The one exception: direct Collison-installation calls with the first 10 users. These are not sales calls — they are research calls where the founder installs the product for the user in real time and watches every point of confusion. No pitch. No deck. No close. See [sales-playbook.md](sales-playbook.md).

---

## Channel Rankings by ROI (Sole Founder, Months 1–18)

| Rank | Channel | Why | Time Investment |
|------|---------|-----|-----------------|
| 1 | **Hacker News** (Show HN, Ask HN) | Highest concentration of early-adopter platform engineers globally. Zero cost. n8n launched here. | 1 post/quarter |
| 2 | **GitHub** (repo + issues + discussions) | `.agent` files in public repos are marketing. Stars are social proof. Issue quality signals product quality. | 10 hrs/wk |
| 3 | **Discord** (AI/agent/compiler communities) | Async, searchable, builder-dense. "Share your pipeline" posts drive organic installs. | 5 hrs/wk |
| 4 | **YouTube** (technical screencasts) | Long tail. "Compiler for agent workflows in 10 minutes" video compounds for 18 months. | 2 videos/mo |
| 5 | **Twitter/X** (build-in-public thread) | Short-form, real-time. "What I learned building a type-checker for LLM tool calls" resonates with PL enthusiasts. | 3 posts/wk |
| 6 | **AWS Marketplace** (Month 12+) | Inbound enterprise buyers with budget. AWS co-sells. Required for VPC deployment Enterprise tier. | 1-time setup |
| 7 | **MCP ecosystem** (Anthropic partner program) | Day-one MCP server + client compatibility puts Conductor in Claude Desktop's extension marketplace. Inbound from Anthropic's community. | 1-time setup |
| 8 | **Newsletter sponsorships** | ByteByteGo, TLDR, The Pragmatic Engineer. Month 9+ when CAC math justifies it. | Paid |

Channels 6–8 are unlocked sequentially: MCP listing (Month 2), AWS Marketplace (Month 12), newsletter sponsorship when LTV/CAC > 3x (estimate Month 10).

---

## Partnership Sequencing

### Phase 1: MCP Ecosystem (Month 1–2)

**Goal**: Appear in every MCP-compatible tool (Claude Desktop, Cursor, Zed) as a first-class citizen.

**Action**:
1. Ship `conductor-mcp-server` — exposes `compile`, `run`, `pipeline_status`, `run_events` as MCP tools.
2. Submit to Anthropic's community MCP server registry (GitHub PR to `anthropics/mcp-servers`).
3. Write a "Conductor + Claude Desktop" getting-started guide; post to HN.

**Expected outcome**: Claude Desktop users can type "run my refactor pipeline" in their AI chat interface and Conductor handles it. This is a zero-cost distribution channel with 50k+ Claude Desktop MAUs.

### Phase 2: AWS Partner Network (Month 6–9)

**Goal**: Get `conductor` listed as an AWS ISV Partner with Bedrock co-sell eligibility.

**Action**:
1. Apply to AWS ISV Accelerate program (requires $1k/mo AWS spend — likely met by Month 4).
2. Write a joint "Conductor + Bedrock" solution brief with AWS Solutions Architect.
3. Co-author a blog post on AWS for Machine Learning blog.

**Expected outcome**: AWS field reps mention Conductor to Bedrock customers building agent workflows. AWS co-sells alongside Bedrock consumption deals. Enterprise tier discovery.

### Phase 3: AWS Marketplace (Month 10–12)

**Goal**: Publish `Conductor Team` and `Conductor Enterprise` as Marketplace listings.

**Action**:
1. Complete AWS Marketplace seller onboarding (SaaS contract type).
2. List Conductor Team ($99/seat/mo) and Enterprise ($2k+/mo custom) as Marketplace products.
3. Enable AWS Private Offers for enterprise negotiations (custom pricing without public listing).

**Expected outcome**: Enterprise buyers can procure Conductor against existing AWS budget commitments. Removes "get a PO approved" friction from enterprise deals.

### Phase 4: Anthropic Partnership (Month 12+)

**Goal**: Co-marketing and technical collaboration with Anthropic on agent workflow tooling.

**Action**:
1. Reach out to Anthropic's developer relations team once 5k GitHub stars and 3 enterprise customers exist (credibility threshold).
2. Propose joint webinar: "Production agent pipelines: from prototype to compiler-checked workflow."
3. Explore being listed in Anthropic's solutions partner directory.

**Expected outcome**: Access to Anthropic's developer audience. Potential featured placement in Anthropic documentation for "how to build production agent workflows."

---

## Launch Week Playbook

Launch Week is a focused 5-day event (Months 3–4, after beta cohort validation) designed to maximize first-week GitHub stars and free sign-ups.

### Day-by-Day Schedule

**Day -7 (Prep)**: README finalized, demo video recorded (< 3 min), landing page live, pricing page live, docs site live, blog post drafted.

**Day 0 (Sunday night 9PM ET)**: GitHub repo goes public. First commit pushed publicly. Star count starts from 0.

**Day 1 (Monday) — HN Launch**

Post to Hacker News at 8AM ET:

```
Show HN: Conductor — a compiled language for LLM agent workflows

I built a compiler for agent pipelines. You write .agent files (think
Terraform but for LLM workflows), the compiler catches tool contract
violations at build time (wrong output type fed to a prompt), and pipelines
run on Bedrock, OpenAI, or self-hosted Gemma4 in one config change.

Example: 5-step code refactoring pipeline that glob()s source files, finds
dead code via Claude, routes to human.checkpoint() for approval, then
applies patches with shell.run().

The compiler gives you:
- Type errors before API calls fire (not 2AM alerts)
- Provider swap in one line (model: claude-3-5-sonnet → gpt-4o)
- GitHub Actions integration (conductor run pipeline.agent)
- MCP server so Claude Desktop can trigger pipelines

GitHub: [link] | Docs: [link] | Free tier: 500 executions/mo

Happy to answer compiler architecture questions — built on top of a
compiler construction course (Jeffery, Packt) that I've been working through.
```

Respond to every comment within 2 hours. No marketing speak in replies — only technical substance.

**Day 2 (Tuesday) — Discord / Communities**

Post in: r/LocalLLaMA, r/MachineLearning (Show HN cross-post style), Discord: Latent Space, AI Engineer, LangChain community, MCP community.

```
Built a type-checked DSL for agent workflows — catches tool contract
violations at compile time instead of 2AM production alerts.

[1-paragraph technical description]
[link to HN post for social proof]
[GitHub link]

Happy to answer questions here or on the repo.
```

**Day 3 (Wednesday) — YouTube**

Publish: "I built a compiler for LLM agent workflows (live demo)"

- 0:00–2:00: Problem (current agent orchestration is untyped Python)
- 2:00–5:00: Write a 5-step pipeline live in the web IDE
- 5:00–8:00: Show the compiler catching a type error (wrong tool output fed to prompt)
- 8:00–10:00: Swap model from Bedrock to self-hosted Gemma4 in one line
- 10:00–12:00: Run in GitHub Actions

Post to YouTube + Twitter/X thread with clips.

**Day 4 (Thursday) — Twitter/X Thread**

"Thread: Why I built a compiler for AI agent workflows instead of another Python library"

1. [Problem] Every agent workflow I've built has the same failure mode: tool output type mismatch at runtime, 2AM alert, manual debugging.
2. [Solution] What if we caught that at compile time? Built a lexer → parser → AST → type checker → IR → runtime for `.agent` files.
3. [Demo clip 1: type error caught at compile time]
4. [Architecture] The compiler maps directly to compiler construction theory: Ch3 lexer → Ch4 parser → Ch5 AST → Ch6 symbol table → Ch7 type checker → Ch8 IR
5. [Demo clip 2: provider swap]
6. [The insight] Provider neutrality is the structural moat. Anthropic can't build a product that routes to OpenAI. We can.
7. [CTA] GitHub: [link] — free tier 500 executions/mo

**Day 5 (Friday) — Retrospective**

Publish blog post: "Launch Week retro: [N] GitHub stars, [N] sign-ups, [N] paying users."

Even if numbers are modest, the retrospective is content that demonstrates transparency and builds trust. n8n's founder did this at every milestone.

---

## First 100 Users Acquisition Path

| Users | Method | Timeframe |
|-------|--------|-----------|
| 1–10 | Collison installation calls with personal network (compiler engineers, AI engineers) | Week 1–3 |
| 10–30 | HN Show HN launch — organic installs from comments | Week 4–6 |
| 30–60 | Discord seeding + YouTube video long tail | Month 2–3 |
| 60–100 | GitHub Stars → word of mouth → organic search | Month 3–4 |

At 100 users with 10 paying ($29/mo Builder): $290 MRR. Infra cost: ~$200/mo. Positive unit economics from user 7.

---

## Metrics to Track Weekly

| Metric | Target (Month 6) | Source |
|--------|-----------------|--------|
| GitHub Stars | 2,000 | GitHub API |
| Free sign-ups | 400 | Postgres |
| Free → Builder conversion | 5% | Stripe |
| Builder MRR | $6,000 | Stripe |
| Weekly active pipelines run | 200 | run_events table |
| p95 compile latency | <500ms | Prometheus |
| NPS (monthly survey) | >50 | Typeform |
