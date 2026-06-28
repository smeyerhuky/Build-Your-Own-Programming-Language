---
type: Business Plan Section
title: "Conductor Labs — Founder Talking Points"
description: Founder talking points for every context and duration. Includes 7-second pitch, 30-second pitch, full 3-minute investor narrative, Why Now and Why You responses, complete objection handling table, and press interview Q&A.
tags: [talking-points, pitch, founder, investor, objection-handling, conductor]
timestamp: 2026-06-28T00:00:00Z
---

# Founder Talking Points

**How to use this document**: Memorize the 7-second and 30-second pitches cold — you need them on demand, with no warm-up, in any social context. Practice the 3-minute pitch until you can deliver it from any starting point (beginning, middle, "just tell me the business model"). Study the objection table until each response is immediate, not retrieved.

The goal is not to sound scripted. The goal is to be so practiced that you can engage with the actual person in front of you — respond to their body language, answer the real version of their question — because the structure is automatic.

---

## 7-Second Pitch

**To a developer**: "It's a programming language for AI agent workflows — your pipelines compile, and type errors show up before any API call fires."

**To an investor**: "Think Terraform but for LLM agent workflows — declarative pipelines with a compiler and type system."

**To a non-technical person**: "When companies use AI to do complex tasks automatically, I built the programming language that makes those AI programs reliable."

**Use case**: party conversations, elevator introductions, "what do you work on?" moments at conferences. The goal is one clean sentence that creates a question, not a complete explanation. The right response from the listener is: "How does that work?" or "Wait, a programming language?" — curiosity, not understanding.

Do NOT try to explain the full value proposition in 7 seconds. Invite the follow-up question.

---

## 30-Second Pitch

"I'm building Conductor — a compiled DSL for agentic coding workflows. The problem is that developers building multi-step LLM pipelines today write Python orchestration code that fails silently and can't be ported between providers. Conductor is a small language — `.agent` files — with a compiler that catches tool contract violations at build time, not at 2 AM. You write the pipeline once, it runs on Claude, GPT-4, or your own Gemma4 GPU, and it runs in CI just like a Terraform plan."

**Use case**: conference introductions, investor cocktail hours, first 30 seconds of any call. This is your invitation to the longer conversation. End it and then stop talking — let the other person ask a question. The most common follow-up is either "How does the type checking work?" (from engineers) or "What's the market look like?" (from investors). Be ready for both.

---

## 3-Minute Pitch (Investor Meeting / Conference Talk)

*Deliver this as continuous narrative — not as bullet points, not from slides. Eye contact throughout. Pause after the "demo moment" to let the visual land.*

---

"Here's the problem I kept running into. I was building multi-step agent pipelines — you know, the kind where you run a code analysis step, pass the output to a refactoring step, then gate on human approval before writing to disk. Standard agentic workflow. I was using LangChain to orchestrate it.

And LangChain is fine for demos. But in production, I kept hitting the same failure mode: a tool call would return the wrong shape of data — say, a list of file paths instead of a structured analysis result — and the next step would just... proceed. No error. The model would do its best with the wrong input and produce subtly wrong output. I'd find out two hours later when a customer hit the broken output.

The root problem is that LangChain is a Python library. It doesn't have a type system. It can't validate that the output of step A is compatible with the input of step B before running anything. That validation only happens at runtime, implicitly, when something breaks in a way that produces a confusing API response instead of an error message.

So I built Conductor — a compiled programming language for agent workflows. You write a `.agent` file — declarative, like a Terraform config — that describes your pipeline steps, what models they use, what tools they call, and how data flows between them. Then you run `conductor compile`. [Pause.] The type checker validates every tool contract before any API call fires. If you try to pipe a `FileList` into a field that expects a `CodeBlock`, you get a build error with a line number. You fix it. Then you run it.

The pipeline runs on AWS Bedrock, OpenAI, or your own self-hosted Gemma4 GPU — the provider is a one-line config change, not a rewrite. And because `.agent` files are text files, they live in your git repo, run in CI, and are reviewed in pull requests like any other infrastructure code.

On the market: developer tools is a $15 billion market growing at 23% CAGR. Cursor showed $2B ARR is achievable in this space. The specific segment I'm targeting — developers building production agent pipelines — didn't really exist 18 months ago. It does now. MCP standardization means every serious AI tool speaks the same protocol. Gemma4 means private GPU serving is now economically viable at $29/mo. And agent workflows have crossed from demo to production — companies are managing fleets of pipelines and discovering they have no tooling for it. That's the market window.

Business model: subscription SaaS. Free tier gets community models. Builder at $29/mo gets Bedrock access and private pipelines. Team at $99/mo per seat gets CI integration and shared workspaces. Enterprise at $2k+ gets SSO, VPC deployment, SLA. Bedrock tokens are billed at cost — zero markup — so Builder gross margin is around 76%.

I'm the founder. 30 years in systems architecture, developer experience, and infrastructure. I built this because I was personally frustrated. I am the target customer.

I'm raising $500k at roughly $4M pre to get to $50k MRR — that's about 170 Builder-equivalent subscribers, achievable in 18 months with the PLG playbook. I'll follow n8n's path: HN Show HN launch, GitHub-first open core, community-driven adoption, founder sells until $1M ARR."

---

*After the 3-minute pitch, stop. The investor will ask questions. The best sign is when they ask about the type system or the IR — they're engaging technically. The second-best sign is when they ask about traction or the market. The concerning sign is when they say "interesting" and pivot to another topic — re-engage with "What would make this more compelling from your perspective?"*

---

## "Why Now?" Response

*This is one of the most common investor questions. Have this fully memorized. It should take 60-90 seconds.*

"Three things converged in 2025-2026 that make this the right moment.

First, MCP. Anthropic's Model Context Protocol has become the de facto tool connectivity standard — Claude Desktop, Cursor, Zed, and dozens of others now speak MCP natively. Conductor is MCP-native from day one, which means we plug into the existing ecosystem without asking anyone to change their stack. Every MCP tool vendor is effectively a Conductor tool provider.

Second, Gemma4 economics. A single A10G spot instance now runs Gemma4-26B at 60 tokens per second, at about five cents per million tokens — versus three to fifteen dollars per million on Bedrock. That's a 60-to-300x cost reduction at 92% quality on code tasks. The private GPU tier, which is critical for regulated industries and is a real pricing differentiator, is only economically viable in 2026. This tier literally didn't make sense 18 months ago.

Third, the demo-to-production transition. Agent workflows crossed from interesting demos to production infrastructure in 2025. Cursor is at $2 billion ARR. Platform engineering teams at serious companies are now writing job descriptions for 'AI Platform Engineer.' They're managing fleets of agent pipelines in production and discovering they have no type safety, no auditability, and no CI story. They need what Terraform gave infrastructure teams in 2016. That market didn't exist 18 months ago — it exists now."

---

## "Why You?" Response

*Delivered conversationally, not as a resume recitation. The goal is to establish that you are uniquely positioned — not that you have impressive credentials.*

"I've spent 30 years in systems architecture, developer experience, testing, and infrastructure. I've seen every version of 'the tooling didn't keep up with the complexity.' I've been the person who had to maintain the system when the tooling was wrong.

I built Conductor because I was personally breaking against LangChain's lack of type safety — repeatedly, in production, in ways that cost real time to debug. I wrote the first version to solve my own problem. That's the product instinct that comes from being your own customer.

The compiler foundation matters. A correct type-checker for agent workflows requires programming language theory that's genuinely rare in the AI tooling space. Most orchestration frameworks are built by ML engineers who know models, not type systems. The Conductor type system handles algebraic data types for tool contracts, context budget tracking as a type constraint, and compile-time MCP schema validation — that's not something you assemble from a Python library in a weekend.

And practically: 74% of YC dev tool companies had purely technical co-founders. This is the right founding profile. The product sells itself when the product works — and I know how to make the product work."

---

## Objection Handling

| Objection | Response |
|---|---|
| **"Why not just use LangGraph?"** | "LangGraph is Python — it's code, not a declaration. You still manage type safety yourself, and you're locked to Python and whatever providers LangGraph supports. Conductor has a real type system and can generate LangGraph code as one of its compilation targets — so if you're already on LangGraph, you can use Conductor as a type-safe frontend." |
| **"Why not use n8n?"** | "n8n is a visual workflow builder — great for ops teams who don't code, wrong abstraction for engineers who do. No loops with back-edges, no git-versionable source, no compile-time type checking. We're building for professional developers; n8n is building for operations teams. Different market." |
| **"Why not just use Claude Projects / AWS Bedrock Agents?"** | "Claude Projects is a chat wrapper. No type system, no CI integration, no multi-provider routing, no private data tier. It's the IDE equivalent of vim without syntax highlighting — it works, but it's not infrastructure tooling." |
| **"Couldn't Anthropic just build this?"** | "Anthropic won't build a product that routes to OpenAI. OpenAI won't route to Anthropic. We can, because we're neutral. That neutrality is the core value. No provider can build the provider-neutral product — that's structurally our lane." |
| **"The `.agent` format is yet another thing to learn."** | "The learning curve is under 10 minutes. The syntax is intentionally minimal — if you can read a GitHub Actions YAML file, you can read a `.agent` file. And the VS Code extension gives you Intellisense and inline type errors, exactly like TypeScript. The onboarding is: install, write one pipeline, run it, see the type checker catch something real." |
| **"What if a big company copies this?"** | "The ecosystem compounds. Once teams have `.agent` files in their repos, CI pipelines, and shared template libraries, the switching cost goes up every week. The compiler expertise took 30 years to build — a team doesn't replicate that in a quarter. And the structural moat (provider neutrality) is one that neither Anthropic nor OpenAI can exploit — they can build a copy but they can't deploy it neutrally." |
| **"This is too early — the agent workflow market isn't big enough yet."** | "Cursor is at $2B ARR selling a $20/mo VS Code extension. The dev tools market willingness-to-pay is proven. The specific segment I'm targeting — production agent pipelines — is the fastest-growing part of that market. I don't need the whole market; I need 500 teams using Conductor at $99/seat." |
| **"Sole founder is a risk."** | "Yes — and the mitigation is that the product doesn't require a large team to reach $1M ARR. PLG means the product acquires users; the founder builds the product. First hire (DevRel) comes at $1M ARR. The architecture, the business plan, and the compiler are all documented such that any future team member can onboard from the wiki in 1-2 days. And 74% of successful YC dev tool companies were built by technical solo founders." |
| **"What if the LLM providers change their pricing and blow up your unit economics?"** | "The credit system is a buffer — Builder users prepay credits, and we reprice credits (not subscriptions) in response to Bedrock price changes. The Gemma4 GPU tier insulates heavy users from Bedrock pricing entirely. And if Bedrock prices drop, our unit economics improve — we pass through at cost, so margin is subscription revenue minus fixed infrastructure." |
| **"How do you prevent a competitor from forking your open-source core?"** | "The open-source core is the compiler and runtime — that's the commodity. The value is in the cloud hosting, team features, GPU serving, and enterprise compliance layer — all closed-source SaaS. We'll evaluate the Business Source License or SSPL to prevent cloud resale, as HashiCorp and MongoDB have done successfully." |

---

## Press Interview Q&A

**Q: What is Conductor?**

A: "It's a programming language for AI agent workflows. Developers write `.agent` files that describe multi-step AI pipelines — what models to use, what tools to call, how data flows between steps. They compile those files, and the compiler catches type errors before any API call fires. Then they run the pipeline anywhere: on AWS, on their own GPU, or in CI."

**Q: Why does this need to be a programming language and not just a library?**

A: "Because agent workflows are software. They have branches, loops, typed state, and contracts between components. Python scripts pretending to be orchestration don't have a type system — errors show up at runtime as confusing API responses, not as 'you passed the wrong type to this step.' A language has a compiler. A compiler validates contracts before anything runs. You can't add that to a library without redesigning the library as a language."

**Q: How does it compare to LangChain?**

A: "LangChain is a Python library — it's great at what it is. Conductor is a language. The comparison is like asking how TypeScript compares to the Express.js library — different abstraction levels, serving different needs. LangChain helps you call LLMs from Python. Conductor is a language where the pipeline itself is the program, with a type system and a compiler. We can generate LangGraph code as a compilation target — LangChain and Conductor aren't necessarily in conflict."

**Q: Who is the target customer?**

A: "Professional developers — senior engineers and platform engineering teams who are running agent pipelines in production and discovering they have no type safety, no CI story, and no provider flexibility. This is not a no-code product; it's infrastructure tooling for developers who write code for a living."

**Q: Why are you the right person to build this?**

A: "I've been the person who maintains the broken pipeline at 2 AM. I built Conductor because I was personally frustrated — I needed exactly this product and it didn't exist. Thirty years in systems architecture and developer experience means I understand both the compiler theory and the DX requirements. And I'm still the target customer — I use Conductor on my own workflows. That's the feedback loop that keeps the product honest."

**Q: What's the business model?**

A: "Subscription SaaS. Free tier gets community models — we want developers to experience the type checker before they pay. Builder at $29/mo is for individual developers who need Bedrock access and private pipelines. Team at $99/mo per seat is for engineering teams with CI/CD integration. Enterprise at $2k+ gets SSO, VPC deployment, and SLA. AWS Bedrock tokens pass through at cost — we don't mark them up. Our revenue is subscription revenue, not token arbitrage."
