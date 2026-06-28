---
type: Business Plan Section
title: "Conductor Labs — Sole Founder Sales Playbook"
description: Sole-founder sales operating manual from Month 1 through $1M ARR. Covers the Collison installation method for first 10 users, HN Show HN launch template, cold outreach scripts, Month 4-6 niche outbound templates, and criteria for hiring the first salesperson.
tags: [sales, playbook, founder-sales, collison-installation, outreach, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Sole Founder Sales Playbook

**Foundational principle** (YC / Garry Tan / Paul Graham, consistent across all batches): "You are the only person capable of selling your product today." The founder IS the sales team until $1M ARR. Not because you cannot afford to hire, but because no salesperson understands the product deeply enough to sell it, no salesperson has your credibility with technical buyers, and the market signal you get from selling personally is irreplaceable for product decisions.

The three rules:
1. Never let a week go by without at least one user call
2. Never delegate demos before $1M ARR
3. Never stop being surprised by what users do with your product

---

## Month 1-2: The Collison Installation Method (First 10 Users)

The Collison brothers (Stripe co-founders) famously set up Stripe for their first customers by sitting down with them and doing the integration in-person, on the spot — instead of sending a link and hoping. This is the Collison installation: don't ask if they want to sign up; ask if you can set it up FOR them.

The method works because:
- It eliminates the "I'll try it later" dropout (later never comes)
- It gives you live observation of where the product is confusing
- It gives the user a working success before they leave the conversation
- It creates a personal relationship that survives early product rough edges

### Who to Target First

Rank candidates by three criteria (in order of importance):
1. **Evidence of pain**: They have publicly expressed frustration with LangChain, LangGraph, or agent orchestration. GitHub issues, blog posts, tweets, Discord messages — any public signal of felt pain.
2. **Seniority**: Senior developer or lead engineer. They have authority to try new tools, they have real production pipelines, and they talk to other senior developers who also need Conductor.
3. **Accessibility**: Twitter/X DM > LinkedIn message > cold email. Warmer is faster. If you have a mutual connection, use it.

**Where to find them**:

- **GitHub**: Search `repo:langchain-ai/langchain label:bug` and `repo:langchain-ai/langgraph issues`. Filter for issues titled "type error", "silent failure", "orchestration", "provider". Extract the issue author's GitHub profile → find their Twitter/LinkedIn.
- **Hacker News**: `hn.algolia.com` → search "LangChain" or "LangGraph" → sort by date (last 6 months) → any comment expressing frustration goes on the list.
- **Twitter/X**: Search query: `(langchain OR langgraph) ("frustrated" OR "broken" OR "maintenance nightmare" OR "rewrite" OR "type error") -filter:replies`. Add authors to list.
- **LangChain Discord**: `#issues-and-bugs`, `#general` — anyone posting about orchestration failures, provider switching, pipeline reliability.

Build a list of 50 names. Rank them. Contact the top 20 in Week 1-2.

### Outreach Script

**Use this for LinkedIn DM, Twitter DM, and email. Adapt the channel-specific conventions (Twitter = shorter).**

---

*LinkedIn / Email version:*

```
Subject: Your [LangChain issue / blog post] on agent orchestration

Hi [Name],

I saw your [GitHub issue #XXX / blog post titled "..." / tweet from last month] about [specific problem: type errors in LangChain/LangGraph, provider lock-in, silent failures].

I'm building Conductor — a compiled programming language for agent pipelines. The core idea: tool contracts are validated at compile time, not at runtime. When you try to pipe a `FileList` into a step that expects a `CodeBlock`, you get a build error — not a confusing API response at 2 AM.

I'd love to set it up FOR you for a workflow you're already building or maintaining. 30 minutes of your time, I do all the work, you tell me where it breaks.

This isn't a sales call — I genuinely want to see how it holds up against a real pipeline, and you get a working setup you can keep using.

Would that be useful?

[Your name]
```

*Twitter/X DM version (shorter):*

```
Hey [Name] — saw your [tweet/reply] about [LangChain orchestration pain].

Building a compiled DSL for agent pipelines — type errors at compile time, not runtime surprises.

Would love to set it up FOR you for a pipeline you're already building. 30 min, I do the work. Worth a try?
```

---

### The "Set It Up FOR Them" Call

Schedule a 30-minute video call. No deck. No slides. Screen-share only.

**Call structure**:

**0-5 min: Understand their current setup**
- "Walk me through the pipeline you're building / maintaining."
- "What's the most painful part of it right now?"
- "Have you hit any type mismatches between steps? What did the error look like?"

Do NOT demonstrate Conductor yet. Listen first. You need their specific pain to make the demo relevant.

**5-20 min: Write their pipeline in Conductor**

Ask them to screen-share their current setup (LangChain code, notebook, whatever). YOU write the `.agent` equivalent in real time while they watch. Narrate as you go:
- "This step is just a tool call — `fs.glob()` → `$source_files`, typed as `FileList`"
- "Now this step takes `$source_files` and passes it to Claude. The type checker will verify `FileList` is compatible with the context field..."
- "Let me add a deliberate type mismatch here — watch what happens when I compile..."

Show the compile error. Show the fix. Show the correct compile. Run the pipeline.

**20-25 min: Validation question**

"Would you pay $29/mo for this vs what you're dealing with today?"

Listen carefully to the answer. If hesitation: "What would need to be different for it to be worth it?" This is your product roadmap signal, not a sales objection to overcome.

If yes: move to next question immediately.

**25-30 min: Convert on the call**

"Can I set you up right now while we're on the call? I'll stay on until you've run your first real pipeline."

Open the signup flow, walk them through it, get their first real pipeline running before the call ends. Do NOT let them leave to "try it later" — that's where users drop off.

### Month 2 Goal

By end of Month 2: 10 users with at least one real pipeline running. 3 paying Builder subscribers ($87/mo total).

Track weekly:
- Calls scheduled this week: target 3-5
- Calls completed: target 3
- Installations completed (user ran a real pipeline): target 2/call
- Conversions to Builder: target 1-2 per 10 users

If conversion rate is below 1 in 10: **product problem**, not sales problem. The pipeline they're building doesn't work in Conductor — fix it.

---

## Month 3-4: HN Show HN + GitHub Launch

This is the single highest-leverage distribution event in Year 1. Timing matters: Wednesday or Thursday, 10am-noon ET (peak HN US traffic).

**Preparation (2 weeks before launch)**:

GitHub repo checklist — do ALL of these before the HN post:
- [ ] Comprehensive README with syntax-highlighted `.agent` example above the fold
- [ ] Clear one-line description in the repo header: "A compiled programming language for AI agent workflows"
- [ ] `/examples/` directory: 5 working pipelines (code review, doc generation, dead code removal, test writer, PR summarizer)
- [ ] One-command install: `pip install conductor-lang` OR `brew install conductor` — your choice, but one command
- [ ] Quick-start that produces visible output in under 5 minutes (time this yourself, fix anything that takes longer)
- [ ] `CONTRIBUTING.md` (signals the project welcomes community; increases star-to-fork ratio)
- [ ] GitHub Actions CI badge in README
- [ ] `v0.1.0` release tag — makes it look like a real project, not a dump of files
- [ ] Issue templates: Bug Report, Feature Request, Pipeline Showcase (the last one invites users to share their pipelines)

### HN Show HN Title Options

A/B test these with 5-10 developer friends before posting:

1. "Show HN: Conductor – a compiled programming language for AI agent workflows"
2. "Show HN: I built a compiler for agent pipelines so type errors happen before API calls"
3. "Show HN: .agent files – write declarative LLM pipelines, compile them, run them anywhere"
4. "Show HN: Conductor – Terraform for AI agent workflows (.agent DSL with a type checker)"

**Recommendation**: Option 2 or 4. Option 2 leads with the personal narrative (HN rewards genuine founder stories). Option 4 leads with the Terraform analogy (investors and tech-savvy HN readers immediately understand the abstraction). Test Option 2 first — if it doesn't top 100 in first hour, you can't change it, but you'll know for next time.

### HN Post Body Template

```
I've been building LangChain/LangGraph pipelines for the past 18 months and the pattern that keeps breaking me is: tool contract violations that only show up as confusing API responses at runtime.

You write a step that pipes `fs.glob()` output into a prompt template. The template expects a code block, not a file list. LangChain doesn't tell you — it just sends the wrong thing to the model and you get a confused response. At 2 AM when the pipeline is running in production.

So I built Conductor — a compiled DSL for agent workflows. You write a `.agent` file declaring your pipeline (steps, tools, context, model routing). The compiler validates tool signatures at compile time: if you try to pipe `FileList` into a field that expects `ExecResult`, you get:

  error[T001]: type mismatch at step FindDeadCode
    context[0]: got FileList, expected ExecResult
    --> src/refactor.agent:12:5

The compiler pipeline is: lexer → parser → AST → symbol table → type checker → IR → runtime. The runtime dispatches to Bedrock, OpenAI, or your self-hosted Gemma4 GPU — provider is swappable in one line.

Here's a complete example — a pipeline that finds and removes dead TypeScript code:

[paste the RefactorDeadCode example from product context]

Try it: [demo link]
GitHub: [link]
Docs: [link]

Happy to answer questions about the language design, type system, or compiler internals.
```

**Engagement rules for launch day**:
- Respond to EVERY comment within 30 minutes for the first 4 hours
- Never be defensive; treat objections as product questions
- Go deep on technical threads — HN rewards genuine technical engagement
- If someone says "why not just use X" — answer it specifically, link to your objection handling (but don't paste a table; write a real response)

### Post-HN Week

Day 1 after HN: Post a Twitter/X thread — "What we learned from 200 developers trying Conductor in 48 hours." Be honest about what surprised you, what broke, what people asked for. This generates a second wave of traffic from people who missed the HN post.

Day 3: Post in LangChain Discord, Anthropic developer community, MLOps Community. Do NOT spam — post genuinely: "I built something related to the LangChain reliability discussion I've seen here. Here's how it handles [specific recurring problem in this community]."

---

## Month 4-6: Outbound for Specific Niches

After the HN launch, organic PLG is running. Use this period to do targeted outbound for three high-value niches:

**Niche 1: Platform engineering teams at mid-size tech companies (50-500 engineers)**
- These teams are explicitly responsible for the reliability of internal tooling including agent pipelines
- They have budget authority for $99/mo Team subscriptions without committee approval
- Find them: LinkedIn search "platform engineer" + "LangChain" OR "agent pipeline" at companies with 50-500 employees
- Entry signal: company's GitHub has LangChain in CI configs, or their engineering blog mentions agent pipelines

**Niche 2: ML engineering teams using LangChain/LangGraph in production**
- These teams are experiencing the pain directly; the lead engineer is your champion
- Find them: GitHub search for repos with >10 stars that use LangChain and have a CI setup
- Entry signal: open issues about orchestration reliability or test coverage for pipelines

**Niche 3: Developer-focused startups where the founder is technical**
- Fastest decision cycle (founder can say yes in 10 minutes)
- Highest NPS potential (founders are motivated users)
- Find them: YC directory, AngelList, Indie Hackers — search for "AI" + "developer tools" founders

### Cold Email Template (use sparingly — max 20/week)

```
Subject: [Company]'s agent pipeline reliability

Hi [Name],

I saw [Company] is using LangChain for [specific use case — pull from their GitHub/blog/LinkedIn, be specific]. 

The pattern I've seen with LangChain orchestration in production: [choose one specific pain that matches their context — silent type failures, no CI integration, or provider switching difficulty].

I built Conductor — a compiled language for agent pipelines. Here's a 90-second demo of it catching a type error that LangChain would have silently failed on: [Loom link — record this for each niche, not a generic demo]

Worth a 20-minute call? I'll bring the same pipeline from your [README / blog post] rewritten in `.agent`, so you can see exactly what changes.

[Your name]
p.s. — if this isn't the right person, can you point me to whoever manages your ML infrastructure?
```

**Cold email standards**:
- Personalization is non-negotiable. Generic emails get ignored. Spend 5 minutes researching before sending.
- The Loom link is mandatory. Text email with no demo gets <1% reply rate. 90-second video showing a real type error gets 10-15%.
- Send from your personal email (`firstname@conductorlabs.com`), not a tool like Outreach. Feels human.
- Max 3 follow-ups, 3 days apart. After that, stop. Do not harass.

---

## Hiring the First Salesperson (At $1M ARR)

**Do NOT hire sales before $1M ARR.** This is a hard rule. Reasons:
- Before $1M ARR, you don't understand why customers buy well enough to train a salesperson
- A salesperson who doesn't understand the product will make commitments the product can't keep
- The market signal from founder-led sales is irreplaceable for product decisions
- Sales compensation at this stage consumes runway without compounding returns

At $1M ARR (~Month 15-18 depending on scenario), hire a **Solutions Engineer / Developer Advocate** — not a traditional Account Executive.

**Why this profile?**

The first "sales" hire at a dev tools company should be someone who can:
- Give a live demo of the product fluently, including edge cases
- Write a `.agent` pipeline on a customer call without preparation
- Explain the type checker to a skeptical senior engineer
- Create technical content (blog posts, YouTube videos, conference talks) that generates inbound

This is a Developer Advocate who can close, not a closer who can do demos.

**Interview process**:

1. **Written screen**: "Explain the difference between a compiled language and a library. Why does it matter for agent orchestration?" — Looking for: technical clarity, ability to communicate to other developers.

2. **Live demo exercise**: Ask them to watch a 10-minute Conductor walkthrough, then give a 15-minute demo of Conductor to an imaginary prospect who is "a senior engineer considering switching from LangGraph." Looking for: fluency, ability to handle objections, genuine enthusiasm.

3. **Pipeline exercise**: Ask them to write a `.agent` pipeline for a simple two-step workflow (code review + summarization). Looking for: can they actually use the product? Do they ask good questions when stuck?

4. **Content exercise**: Ask them to propose a 3-month content calendar for Conductor's blog. Looking for: understanding of the developer audience, ability to map technical features to developer pain points.

**Compensation at $1M ARR** (seed-funded):
- Base: $130-150k
- Variable: $20-30k tied to MAU growth (not ARR — in Year 1, community building matters more than closing deals)
- Equity: 0.5-1% (4-year vest, 1-year cliff)
- Success metrics for first 90 days: 1k Discord members, 2k GitHub stars, 3 conference talks scheduled, 5 community pipeline templates submitted

**What they should NOT do**:
- Cold outbound (you've built an inbound machine — protect it)
- Manage enterprise sales cycles (that comes with the Solutions Engineer hire at $3M ARR)
- Create paid advertising (still PLG-only)
