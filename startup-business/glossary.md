---
type: Glossary
title: "LangCraft Inc. — Business Glossary"
description: Definitions for business, product, sales, and finance terms used throughout the startup-business bundle.
tags: [okf, glossary, startup-business, langcraft]
timestamp: 2026-06-28T00:00:00Z
---

# Business Glossary

Definitions used throughout the `startup-business/` bundle. Terms are listed alphabetically.

---

## A

**ARR** — Annual Recurring Revenue. The annualised value of all subscription contracts. Formula: `MRR × 12`.

**AST** — Abstract Syntax Tree. A tree representation of source code after parsing; a core J0 compiler data structure. Also used loosely in product docs to mean the structured representation of a customer's domain model.

---

## B

**Bootstrapped** — A company funded entirely by its founders and revenue, without external investment.

**Burn rate** — Monthly net cash outflow (expenses minus revenue). "Runway" equals cash on hand divided by burn rate.

**Bytecode** — Platform-independent binary instruction set. LangCraft's J0 bytecode is the 23-opcode `.j0` format defined in [RFC-002](./rfc/rfc-002-bytecode-format.md).

---

## C

**CAC** — Customer Acquisition Cost. Total sales and marketing spend in a period divided by new customers acquired in that period.

**Churn** — The rate at which customers cancel subscriptions. Monthly churn = customers lost in month / customers at start of month.

**Compiler pipeline** — The sequence of transformations from source text to executable output: lexer → parser → AST → symbol table → type checker → TAC → codegen.

**Conversion rate** — Percentage of prospects who move from one funnel stage to the next (e.g., trial → paid).

---

## D

**DAU/MAU** — Daily Active Users / Monthly Active Users. The ratio measures engagement; >0.2 is healthy for a developer tool.

**Delaware C-Corp** — A U.S. corporation incorporated in Delaware; the standard legal structure for venture-backed startups.

**DSL** — Domain-Specific Language. A programming language designed for a specific problem domain (e.g., configuration, query, workflow). LangCraft's primary commercial use case.

---

## E

**Enterprise tier** — The highest-priced subscription tier, sold to organisations with >50 engineers; includes SSO, audit logs, SLA, and dedicated support.

---

## F

**Freemium** — A business model where a basic product is free and advanced features require payment.

**Funnel** — The stages a prospect moves through: Awareness → Interest → Evaluation → Purchase → Expansion.

---

## G

**Gross margin** — Revenue minus cost of goods sold (COGS), expressed as a percentage. SaaS gross margins typically run 70–85 %.

**GTM** — Go-to-Market. The strategy for how a product reaches its target customers.

---

## I

**ICP** — Ideal Customer Profile. A precise description of the company, team, and individual most likely to buy and succeed with the product. See [`customer/icp.md`](./customer/icp.md).

**J0** — The small Java subset taught in the *Build Your Own Programming Language* course. J0 supports classes, methods, fields, arithmetic, comparisons, loops, and I/O. It is the reference language for LangCraft's toolchain.

---

## L

**LangCraft** — The startup commercialising the J0 toolchain and compiler-education platform.

**LSP** — Language Server Protocol. A standard JSON-RPC protocol allowing editors (VS Code, Neovim, etc.) to query a language server for hover info, diagnostics, go-to-definition, and code completion.

**LTV** — Lifetime Value. The total net revenue expected from a single customer over the full duration of their relationship. Formula: `ARPU × Gross Margin % / Churn Rate`.

**LTV:CAC ratio** — The ratio of lifetime value to acquisition cost. Target ≥ 3:1 for a healthy SaaS business.

---

## M

**MRR** — Monthly Recurring Revenue. The sum of all active subscription revenue normalised to one month.

**MVP** — Minimum Viable Product. The smallest version of a product that delivers core value and can be shipped to real users for feedback.

---

## N

**NPS** — Net Promoter Score. A customer satisfaction metric: % promoters minus % detractors. Scores above 40 are excellent for a developer tool.

**NRR** — Net Revenue Retention. Revenue from the existing customer base at end of period divided by revenue at start of period. >100 % means expansion outweighs churn.

---

## O

**OKF** — Open Knowledge Format. A lightweight file-and-frontmatter convention used in this repository to make documents navigable by both humans and LLMs.

**OKR** — Objective and Key Result. A goal-setting framework: one *objective* (qualitative direction) plus 2–5 *key results* (measurable outcomes). See [`ops/okrs.md`](./ops/okrs.md).

---

## P

**Payback period** — Months of revenue required to recover the cost of acquiring a customer. Target <12 months for SMB, <18 months for enterprise.

**PLG** — Product-Led Growth. A go-to-market motion where the product itself drives user acquisition, conversion, and expansion (as opposed to a sales-led motion).

**Positioning** — How a product is defined relative to alternatives in the minds of its target buyers. See [`sales/positioning.md`](./sales/positioning.md).

**PRD** — Product Requirements Document. Specifies what a product must do, for whom, and how success will be measured. See [`product/prd.md`](./product/prd.md).

**PRFAQ** — Press Release / FAQ. An Amazon-style document that describes the product from the customer's point of view, written as if it has already shipped. See [`product/prfaq.md`](./product/prfaq.md).

**Pre-seed** — The earliest funding round, typically $250K–$2M, used to validate a hypothesis before building a full product.

---

## R

**RFC** — Request for Comments. A numbered technical specification document. See [`rfc/process.md`](./rfc/process.md) for the LangCraft RFC lifecycle.

**Runway** — How many months a company can operate at its current burn rate before exhausting cash.

---

## S

**Seed** — An early funding round, typically $1M–$5M, used to build a product and find initial customers.

**SMB** — Small and Medium Business. Companies with 10–500 employees; a secondary target segment for LangCraft.

**SaaS** — Software as a Service. A subscription-based software delivery model hosted in the cloud.

---

## T

**TAC** — Three-Address Code. The intermediate representation used in the J0 compiler backend (Ch 8); also acronym for Total Addressable Cost in some finance contexts.

**TAM** — Total Addressable Market. The total revenue available if a product captured 100 % of its market.

**Toolchain** — The complete set of tools required to compile and run a language: lexer, parser, compiler, assembler/VM, debugger, IDE plugin.

---

## U

**Unit economics** — The revenue and cost associated with a single unit of business (e.g., one customer). See [`finance/unit-economics.md`](./finance/unit-economics.md).

---

## V

**VC** — Venture Capital. Professional investment firms that fund high-growth startups in exchange for equity.

---

## W

**WAU** — Weekly Active Users. A middle-ground engagement metric between DAU and MAU.
