---
type: Business Plan Section
title: "Conductor Labs — Customer Acquisition Playbook"
description: Week-by-week customer acquisition playbook from first 10 users through first enterprise contract. Covers list-building methodology, Discord and YouTube strategy, Launch Week runbook, and enterprise champion identification script.
tags: [customer-acquisition, growth, plg, discord, launch-week, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Customer Acquisition Playbook

**Philosophy**: Every acquisition tactic in this document is either free or requires only founder time. No paid advertising until $1M ARR. Every feature is a distribution unlock. Every piece of content is a customer acquisition event.

The n8n benchmark: Jan Oberhauser (sole founder, similar PLG playbook) reached $6M ARR before any institutional capital. The path is: personal outreach (10 users) → HN Show HN (100-500 signups) → content compounding (1k users) → community flywheel (10k users) → enterprise discovery (first $2k+/mo contract).

---

## First 10 Users (Month 1, Weeks 1-8)

### Week 1-2: Build the Target List

The goal is 50 high-quality names ranked by pain signal, seniority, and accessibility. Do not skip this step — targeting matters more than volume.

**Source 1: LangChain / LangGraph GitHub**

```bash
# Search issues with these labels/keywords:
# - Type error, silent failure, orchestration, provider switching
# - Repos: langchain-ai/langchain, langchain-ai/langgraph
```

Go to `github.com/langchain-ai/langchain/issues` and search: `orchestration type error`. Also search `silent failure`, `pipeline reliability`, `provider lock-in`. Export: issue author, date, GitHub profile URL.

For each GitHub profile: check if they have a Twitter/X link or personal website. Priority: people with public profiles (easier to contact) who have more than one activity signal (opened an issue AND have a blog post or tweets about the same topic).

**Source 2: Hacker News**

Go to `hn.algolia.com`. Search: `"LangChain" frustration` (last 6 months). Also search: `"LangGraph" production`, `"agent pipeline" reliability`. Any comment with genuine frustration (not trolling) — add the author to the list. Note the username; use `hn.algolia.com/user/[username]` to find their linked site/Twitter.

**Source 3: Twitter/X**

Search query: `(langchain OR langgraph) ("frustrated" OR "broken" OR "maintenance nightmare" OR "rewriting" OR "type error" OR "silent fail") -is:retweet lang:en`

Date filter: last 6 months. Add any author who sounds like a developer with production problems (not academic researchers, not "I tried LangChain for 10 minutes" beginners).

**Source 4: LangChain Discord**

Join the LangChain Discord (public). Browse `#issues-and-bugs` and `#general`. Anyone posting about orchestration failures, type mismatches, provider switching, or "should I switch to LangGraph" — add to list.

**Ranking criteria**:

Score each name 1-3 on each axis:
- Pain clarity (1 = vague complaint, 3 = specific technical problem with details)
- Seniority signal (1 = student/hobbyist, 3 = senior engineer / platform lead)
- Accessibility (1 = email only, 2 = LinkedIn, 3 = Twitter/X with recent activity)

Sort descending by total score. Your top 20 get contacted in Week 3.

### Week 3-6: Collison Installations

Using the outreach scripts in [sales-playbook.md](sales-playbook.md): contact your top 20 targets. Goal is 10 calls.

**Tracking sheet** (maintain this religiously):

| Name | Company | Source | Contacted | Response | Call scheduled | Pipeline installed | Paying |
|---|---|---|---|---|---|---|---|
| — | — | — | — | — | — | — | — |

Update this after every interaction. The first sign of product-market fit is: did they return after the first call? Did they run a second pipeline without you prompting? If not — why not? Call them and ask.

**The most important question to ask every user who doesn't convert**:

"You tried it, you saw it work — what would have to be true for you to use it weekly?"

The answer to this question is your product roadmap for Month 2.

### Week 7-8: First Revenue

By the end of Week 6, you should have 10 users with at least one pipeline running. Now: follow up with each personally.

Template (email or DM):

```
Hey [Name],

Checking in on how Conductor is holding up for your [describe their specific pipeline use case].

Have you run any pipelines this week?

If you have and it's been useful — I'd love to set you up on Builder ($29/mo) so you get Bedrock access and private pipelines. Can do it in 2 minutes while we're talking.

If you haven't — totally fine to tell me why. What would make it useful enough to run weekly?

[Your name]
```

Goal: 3 Builder subscribers ($87/mo total) by end of Month 2.

If you have fewer than 3 paying users by end of Month 2: this is a product problem. Do not proceed to the HN launch. Spend one more month on Collison installations and product iteration. The HN launch is a one-time event — don't waste it on a product that isn't landing.

---

## First 100 Users (Month 3-4)

### Primary: HN Show HN Launch

See [sales-playbook.md](sales-playbook.md) for the complete HN Show HN playbook, post template, and launch timing guidance.

**Expected outcomes from HN Show HN**:
- If post reaches top 5 on Show HN: 200-500 signups in 72 hours
- If post reaches top 10: 100-200 signups
- If post doesn't crack top 10: post a comment with a substantive update 24 hours later; this often resurfaces posts

Conversion expectation: 5-10% of signups try the product. 2-5% of signups convert to Builder within 30 days (based on comparable dev tool PLG benchmarks).

### Secondary Channels (Run Concurrently with HN)

**GitHub repository launch** (same day as HN post):
- Ensure repo is public before HN post goes live
- Post a pinned issue titled "Launch day: community Q&A — ask me anything about the type system, compiler, or language design"
- Respond to every GitHub issue and PR within 24 hours for the first week

**LangChain Discord post**:
```
Hey all — I built something I've been wanting to exist for a while: a compiled DSL for agent pipelines with a real type checker.

The motivation was exactly the problem I kept seeing discussed here: tool contract violations that only show up as confusing API responses. I wanted compile-time errors instead.

Here's the GitHub if you want to look: [link]

Happy to answer questions here or jump on a call if you want to try it on a pipeline you're already building.
```

Post this in `#show-off-your-projects` (get community manager permission first — most active Discords have channel rules). Do NOT post in `#issues-and-bugs` or `#general` without it being relevant to an active conversation.

**Anthropic developer community** (forum / Discord): same copy, adapted to highlight MCP compatibility — that's the most relevant hook for the Anthropic community.

**Twitter/X thread** (post 2 hours after HN to let HN momentum build):

```
Thread: Why I built a programming language for AI agent workflows (and what I learned building the compiler)

1/ The failure mode that drove me crazy with LangChain: tool contract violations that only show up as confusing API responses. No error message. Just a wrong result at 2 AM.

2/ So I built @ConductorLang — a compiled DSL for agent pipelines. Write a .agent file, compile it, type errors show up before any API call fires.

3/ Here's what that looks like in practice: [screenshot of type error message with line number]

4/ The pipeline syntax is intentionally minimal. If you can read a GitHub Actions YAML, you can read a .agent file: [code screenshot]

5/ The compiler pipeline is: lexer → parser → AST → symbol table → type checker → IR → runtime. The IR dispatches to Bedrock, OpenAI, or self-hosted Gemma4. Provider is a one-line change.

6/ MCP-native from day one — any MCP tool is immediately available as a first-class tool in your pipeline. No wrapper, no adapter needed.

7/ Run in GitHub Actions: `conductor run` is a drop-in CI step. Pipeline as infrastructure, not as a script.

8/ GitHub: [link] | Demo: [link] | Docs: [link]

Questions? Reply here or check the HN thread: [link]
```

### Month 3-4 Content Calendar

Content is customer acquisition. Every piece of content is a potential first touchpoint for a developer who searches for "LangChain alternative" or "agent pipeline type safety."

| Week | Content | Format | Platform |
|---|---|---|---|
| 3.1 | HN Show HN + GitHub launch | Post | HN + GitHub |
| 3.2 | "How the Conductor type checker works" | Technical blog | Blog + HN Ask |
| 3.3 | "We replaced 800 lines of LangChain code with a 60-line .agent file" | Case study | Blog + Twitter |
| 3.4 | "Conductor vs LangGraph: a direct comparison" | Technical post | Blog + LangChain Discord |
| 4.1 | VS Code extension launch | Product post | Twitter + Product Hunt |
| 4.2 | "Running Conductor in GitHub Actions" | Tutorial | Blog + GitHub |
| 4.3 | Community pipeline showcase #1 | Feature | Discord + Twitter |
| 4.4 | "What we learned from 200 developers" | Reflection | HN + Blog |

**SEO targets** (write content that matches these search queries):
- "langchain alternative" — currently low competition, growing volume
- "agent pipeline type safety" — low competition, perfect intent match
- "compiled llm orchestration" — very low competition, novel category
- "agent workflow ci cd" — medium competition, high intent

---

## First 1,000 Users (Month 5-6)

### Discord Community Strategy

**Set up the Conductor Discord server in Month 2** (before the HN launch — you want it ready when traffic hits). Do not wait until you need it; by the time you need it, it's too late to build the culture.

Channel structure:
- `#announcements` — Conductor releases, blog posts, important updates (founder only)
- `#general` — community conversation
- `#show-your-pipelines` — developers share `.agent` files they've written (most important channel)
- `#help` — troubleshooting, type checker questions, tool integration
- `#feature-requests` — community-sourced roadmap (this is product research)
- `#off-topic` — general dev conversation, keeps people around

**Community management rules** (founder enforces personally until Month 13):
- Respond to every `#help` question within 2 hours — every time. This is the commitment that builds trust.
- Post in `#show-your-pipelines` first: share your own `.agent` pipelines regularly. The founder modeling behavior encourages community members to do the same.
- React to every pipeline posted in `#show-your-pipelines` with a substantive comment (not just an emoji). "This is a clever use of the branch construct — did you consider..." signals that the founder is engaged.
- Never delete negative feedback. Address it publicly. "This is a fair point — here's how we're thinking about it, and here's the GitHub issue if you want to track progress."

**Discord growth targets**:
- Month 3: 100 members (HN launch drives signups)
- Month 4: 250 members
- Month 5: 500 members
- Month 6: 1,000 members

A Discord member who posts in `#show-your-pipelines` is a near-certain paying customer or a powerful referral source. Track this conversion rate.

### YouTube Demo Strategy

Three videos, done by founder, screen recordings + voiceover. Production quality: good audio, no need for camera, clean screen. Time investment: ~4 hours per video including recording and basic editing.

**Video 1**: "Write your first Conductor pipeline in 10 minutes"
- Start from scratch: `pip install conductor-lang`
- Write a 2-step code review pipeline
- Deliberately introduce a type error and show the compiler catching it
- Run the pipeline on a real TypeScript file
- Title optimization: "LangChain type errors explained + how to fix them with a compiled DSL"

**Video 2**: "Conductor vs LangChain: type safety demo"
- Side-by-side: the same pipeline in LangChain Python (left) and Conductor `.agent` (right)
- Introduce the same type error in both
- Show: LangChain produces wrong output silently; Conductor refuses to compile
- Title optimization: "Why LangChain doesn't tell you about type errors (and what does)"

**Video 3**: "Running Conductor in GitHub Actions"
- Set up `conductor/run-agent@v1` in a real repo
- Show the CI check failing on a type error
- Show it passing after fix
- Show the pipeline output in the GitHub Actions log
- Title optimization: "AI agent workflow CI/CD: how to test your agent pipeline in GitHub Actions"

Post all three in Week 1 of Month 5. Add to the GitHub repo's README under "Tutorials."

### Launch Week Playbook (Month 6)

Modeled after Supabase's "Launch Week" format — a themed week of daily releases that creates a compounding attention spike.

**Announce the Launch Week 2 weeks in advance** (Twitter + Discord + email list).

Day 1 (Monday): **Conductor Launch Week announcement**
- Tweet: "This week is Conductor Launch Week. Five days, five releases. Here's what's coming..."
- Email to beta user list: "You helped us get here. Launch Week starts Monday."
- Discord: pin the announcement

Day 2 (Tuesday): **"How the compiler works" — deep technical blog post**
- Walk through the complete compiler pipeline (lexer → parser → AST → type checker → IR → runtime) with annotated source code examples
- This is your credibility post — PL-theory literate developers will share this
- Submit to HN as "Ask HN: how we built the type checker for an AI agent workflow language"

Day 3 (Wednesday): **5 new community templates launch**
- Pre-write 5 templates in the week before Launch Week
- Release them all on Day 3: code review, documentation generation, PR summarization, dead code analysis, test writing
- Community challenge: submit your own template, featured in next month's release

Day 4 (Thursday): **"Conductor + AWS Marketplace" announcement**
- If Marketplace listing is live: announce it
- If not yet live: announce "Conductor is now AWS ISV Accelerate qualified" (which is the step before Marketplace listing)
- Include: how enterprise teams can deploy Conductor in their own VPC

Day 5 (Friday): **AMA + community showcase**
- Live AMA on Discord at 1pm ET: "Founder AMA — anything about the language design, type system, compiler, or roadmap"
- Feature 3 community pipelines submitted during Launch Week
- Announce Month 7 feature: what comes next

**Expected results from Launch Week**:
- +500 Discord members
- +1,000 GitHub stars
- 5-10 new Builder subscribers
- 2-3 press/blog mentions from developer media following the technical posts

---

## First Enterprise Customer (Month 7+)

### Bottom-Up Champion Identification

Enterprise sales at this stage is never top-down. It is always: individual developer discovers Conductor → uses it personally → brings it to their team → team uses it → platform engineering lead decides to formalize.

Identify champions proactively:
- Set up usage alerts: any user who runs >100 pipelines/month on Team tier is a signal. Create a Slack/email alert for this threshold.
- Monitor company email domains on Builder/Team tier. If you see 3+ people from `@company.com`, look up the company. Find the platform engineering or ML infrastructure lead.
- GitHub signal: if a company's public repos include `.agent` files or reference Conductor in CI configs — that's enterprise discovery happening organically.

**Champion outreach script** (warm — they're already using Conductor):

```
Subject: [Company]'s Conductor usage + enterprise tier

Hi [Name],

I noticed [Company] has several developers running Conductor pipelines — that's great to see. Thank you for being early users.

I wanted to reach out personally because the usage patterns suggest you might be hitting the limits of the Team tier (especially around [shared secrets / audit logs / VPC isolation / SSO]).

I'd love to set up a quick call to understand what your team is building and whether the Enterprise tier would remove any friction. I can have a trial environment configured in your own AWS account within a week.

Happy to do this as a no-commitment technical trial first.

[Your name], Conductor Labs
```

### Enterprise Deal Process

The five-stage motion for first enterprise contracts:

**Stage 1: Champion introduction (Day 1)**
- 30-minute call with the champion (the developer who brought Conductor in)
- Goal: understand their use case, team size, and the organizational friction they're experiencing
- Question: "What would make Conductor indispensable to your team — what's currently in the way?"

**Stage 2: Technical POC (Week 1-2)**
- You personally set up Conductor in their environment: their AWS account, their VPC (if they need it), their CI system
- Run one real pipeline from their production stack
- This is the Collison installation, enterprise edition
- Success criteria: one real pipeline running in their environment before end of POC

**Stage 3: Security review (Week 2-3)**
- Provide SOC2 Type I report (have this done by Month 5)
- Demo SAML SSO with their identity provider (Okta, Azure AD)
- Answer their security questionnaire — have a template pre-filled based on common questions
- Common blockers: data residency, access logging, penetration test report — have these ready

**Stage 4: Commercial (Week 3-4)**
- Standard Enterprise: $2,000/mo for up to 10 seats, unlimited executions, VPC deployment
- Annual contract preferred (offer 2 months free on annual vs monthly)
- Net 30 payment terms as default; push for net 15 if their procurement allows
- Do NOT negotiate on price before you have 3+ enterprise customers — you don't know your floor yet

**Stage 5: Reference and case study**

After 60 days of active use, ask:
```
"We're preparing a case study about how [Company] uses Conductor. Would you be willing to be featured? It would include: what you were using before, what you migrated, and what the outcome was — all numbers reviewed by you before publication."
```

Case study template:
> **[Company] runs [N] agent pipelines in production across [N] team members. Before Conductor, they spent [X hours/week] debugging silent type failures in their [LangChain / LangGraph] orchestration. After migrating [N] pipelines to Conductor: [specific outcome — time to debug reduced by X%, pipeline failures in production reduced from Y to Z, type errors caught at compile time: N].**

A case study from a named company is worth 20 blog posts. Prioritize getting this written.
