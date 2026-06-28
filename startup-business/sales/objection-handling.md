---
type: Objection Handling
title: "LangCraft — Sales Objection Handling"
description: Top 10 sales objections heard from prospects and the recommended responses for each.
tags: [objections, sales, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Objection Handling

Top 10 objections encountered in sales conversations, with recommended responses.

---

## 1. "Why not just use ANTLR?"

**Who says this:** Enterprise DSL builders and experienced engineers.

**The real concern:** They already know about ANTLR and wonder if we're reinventing it.

**Response:**

> "ANTLR is a fantastic parser generator and we respect it deeply. But ANTLR covers one phase of a compiler — parsing. It doesn't teach or provide AST construction, symbol tables, type checking, code generation, or a bytecode VM. And it has no educational curriculum, no cloud sandbox, and no LLM integration. LangCraft covers the *complete* compiler pipeline. If you've already got ANTLR in your stack, LangCraft may actually tell you what to do with the parse tree it produces."

**Proof point:** Show the pipeline diagram (source → tokens → AST → TAC → VM → native) and highlight everything that comes after parsing.

---

## 2. "Why would I pay for this when I can just buy the book?"

**Who says this:** Self-directed individual learners.

**The real concern:** Price sensitivity; unclear differentiation from the book.

**Response:**

> "The book is fantastic and I wrote it. But let's be honest about what a book can't do: it can't run your code, tell you what went wrong, or answer your question at 11pm when you're stuck. LangCraft does all three. The free tier is genuinely free — you can go through all five phases, run all the exercises, and get LLM help. If the book is what got you started, LangCraft is what will get you finished."

**Proof point:** Free tier CTA — "Just try it. It's free and takes 5 minutes."

---

## 3. "This is too basic for us — we need a production-grade language, not a teaching language."

**Who says this:** Enterprise DSL builders who want a production system.

**The real concern:** J0 is a learning language; they need something battle-tested.

**Response:**

> "J0 is a teaching language, but the toolchain architecture is production-grade. J0 was designed so that every component — the lexer, parser, AST, type system, bytecode format — is a direct analog of what you'd build in a production DSL. Enterprise customers use LangCraft to *learn the architecture* and then extend it to their use case. With the enterprise tier, you get a VS Code plugin, LSP server, and CI/CD integration that you can adapt to your grammar. Think of J0 as the scaffold you replace with your own grammar as you go."

**Proof point:** Show the RFC-001 J0 spec and RFC-004 LSP integration doc — these are real specifications.

---

## 4. "Our security team won't allow cloud-based code execution."

**Who says this:** Enterprise prospects at regulated companies (financial services, defense, healthcare).

**The real concern:** Source code confidentiality; compliance requirements.

**Response:**

> "We hear this often and it's completely valid. That's exactly why we're building a self-hosted option in our Enterprise tier. Your source code never leaves your network — LangCraft runs in your VPC in a Docker container. The self-hosted option is on our Q2 2027 roadmap. If this is a blocker for you today, we'd love to put you on our early access list for the self-hosted beta and offer a pilot at no cost in the meantime."

**Proof point:** Show the self-hosted roadmap item (B-050 in feature-backlog.md). If asked for a timeline, say Q2 2027 — offer to accelerate based on customer commitment.

---

## 5. "We tried to learn compiler construction before and it was too hard."

**Who says this:** Individual learners and enterprise teams who have burned time on this.

**The real concern:** Wasted effort; scepticism that anything will be different.

**Response:**

> "We built LangCraft specifically because of stories like yours. The reason it was hard is that you were doing it alone, without feedback, with an environment that probably took a day to set up. LangCraft removes every one of those barriers. You're in the sandbox in 5 minutes. When you hit an error, the LLM assistant explains it in plain English. The exercises check your work automatically. We've had users who failed at self-directed compiler learning three times before and completed Phase 1–3 on LangCraft in a single weekend."

**Proof point:** NPS testimonials; show the onboarding flow live.

---

## 6. "We already have JetBrains MPS."

**Who says this:** Enterprise teams at large companies (often in Germany, the Netherlands, or Scandinavia where MPS has stronger adoption).

**The real concern:** They've invested in MPS and don't want another platform.

**Response:**

> "MPS is powerful for projectional editing use cases — if you're building a language with a non-textual representation, it's genuinely the best tool for that. But for teams building text-based DSLs who want a fast, learning-friendly, LSP-compatible toolchain, LangCraft is a very different product. If your team has struggled to onboard new engineers to MPS, LangCraft might be the training layer that actually gets them productive. It doesn't have to be either/or."

**Proof point:** Competitive analysis — MPS average onboarding time vs. LangCraft.

---

## 7. "What happens to my work if LangCraft shuts down?"

**Who says this:** Individual learners and cautious enterprise buyers.

**The real concern:** Vendor lock-in; company longevity risk.

**Response:**

> "Great question and a fair concern for any SaaS product. Two things: first, your source code is always exportable from the sandbox as a ZIP file — you own it completely. Second, the J0 compiler itself is MIT-licensed open source. Even if LangCraft ceased to exist tomorrow, the toolchain would live on in the GitHub repository. You can clone it, run it locally, and teach it however you want. We're building a cloud product on an open-core foundation precisely because we believe in not locking people in."

**Proof point:** MIT license link; export feature in the product.

---

## 8. "Can't ChatGPT / GitHub Copilot just do this for me?"

**Who says this:** Savvy engineers who are power LLM users.

**The real concern:** Why pay for LangCraft if a general-purpose LLM can help?

**Response:**

> "It's a good question. We've tried it. General-purpose LLMs are great at explaining concepts but poor at guiding you through a *structured curriculum* on a *specific codebase*. They hallucinate JFlex options, suggest BYacc flags that don't exist, and can't tell you which exercise you're on or what the automated test expects. LangCraft's LLM assistant is trained on the specific OKF course content and knows the exact state of your sandbox. It's the difference between asking a stranger for directions and having a guide who knows the route."

**Proof point:** Side-by-side demo: ask ChatGPT and LangCraft assistant the same question about a compiler error.

---

## 9. "The pricing seems high for what is essentially a course platform."

**Who says this:** Budget-conscious individual learners or SMB buyers.

**The real concern:** Perceived value vs. price; comparing to $10 Udemy courses.

**Response:**

> "I understand the comparison. Here's the difference: a $10 Udemy course gives you video content. LangCraft gives you a live sandbox, automated exercise grading, an LLM assistant, a VS Code extension, and a portfolio certificate. The relevant comparison is not Udemy — it's a $500 MOOC from Coursera, or the $30K you'd spend hiring an engineer to figure this out without the platform. Pro at $29/month works out to less than $1/day. And the free tier is genuinely free — you can complete the whole curriculum without paying anything."

**Proof point:** Show value-per-feature table from pricing doc.

---

## 10. "We need to run this by our legal / procurement team."

**Who says this:** Enterprise buyers at larger companies.

**The real concern:** This is often a stall, not a real objection. Sometimes it's genuine process.

**Response:**

> "Of course — that's a completely normal part of enterprise evaluation. To make it as easy as possible: we have a standard MSA, a DPA for GDPR compliance, and we can provide our SOC 2 Type II report under NDA. If procurement needs a vendor questionnaire, we've pre-filled the most common ones. What's the typical timeline for your procurement process? I can set up a call with your legal team if that helps move things along."

**Proof point:** Have the MSA template, DPA, and security questionnaire pre-prepared. Speed of response to legal requests is itself a trust signal.
