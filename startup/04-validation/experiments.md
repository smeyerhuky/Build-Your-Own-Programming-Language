---
type: Validation
title: "Pre-Build Validation Experiments — Conductor Labs"
description: Five time-boxed experiments to run before any Phase 1 code is written — customer discovery interviews, syntax usability testing, pricing sensitivity, distribution channel validation, and GPU model quality benchmarking.
resource: /startup/04-validation/experiments.md
tags: [validation, experiments, customer-discovery, syntax, pricing, distribution, benchmark]
timestamp: 2026-06-28T00:00:00Z
---

# Pre-Build Validation Experiments

Five experiments to run **before writing a single line of Phase 1 production code**. Each experiment is time-boxed, has a clear hypothesis, a specific method, a binary success criterion, and a decision rule for what to do with the results.

**Total budget**: ~40 hours of founder time, ~$200 in infrastructure costs.  
**Total timeline**: 2 weeks (Experiments 2, 3, and 4 run concurrently in week 2; Experiment 5 runs in 3 days).

---

## Experiment 1: Customer Discovery Interviews

**Hypothesis**: Senior developers actively using LangChain or LangGraph feel significant pain from the lack of type safety, provider portability, and execution auditability — and at least half would pay $29/mo for a solution.

**Method**: 10 interviews with active LangChain/LangGraph users (GitHub issues, Discord, Twitter). Follow strict script: surface pain before mentioning any product. Ask willingness-to-pay only at the end.

**Interview guide** (strict order):
1. Walk me through a typical agent pipeline you've built recently.
2. What breaks most often? Tell me about the last production failure.
3. How do you handle multiple LLM providers? Have you ever needed to switch?
4. When you look at your pipeline code six months later, can you understand it? Can someone else?
5. How confident are you that your tool contracts are correct before running?
6. If a compiler checked tool contracts at build time, what would that be worth monthly?

**Success criteria**: ≧7 of 10 interviews surface pain unprompted; ≥5 of 10 willing to pay $29/mo; ≥3 commit to beta.

**Decision**: <7 surface pain → DO NOT proceed to Phase 1. Pivot or shut down.

---

## Experiment 2: Prototype Syntax Usability Test

**Hypothesis**: Developers familiar with Python and YAML can produce a syntactically valid `.agent` pipeline without documentation in under 10 minutes.

**Method**: 5 developers from Exp 1 volunteers. Give blank `.agent` template and task description ("build a pipeline that: globs TS files, asks AI to find unused functions as JSON, loops to remove each one"). Observe silently. Time to first valid structure.

**Success criteria**: ≤4 of 5 produce a pipeline with correct top-level structure, a loop with iteration variable, and at least one tool call within 10 minutes.

**Decision**: <3 pass → syntax redesign required before lexer/parser implementation.

---

## Experiment 3: Pricing Sensitivity Test (Van Westendorp)

**Hypothesis**: $29/mo falls within the acceptable range for the Builder tier; $99/seat is "getting expensive but acceptable" for engineering teams.

**Method**: 5-question Typeform survey (Van Westendorp PSM) distributed to Exp 1 subjects + LangChain Discord + AI communities. Target 30 responses. Questions: too cheap / bargain / getting expensive / too expensive at various price points; role self-identification.

**Success criteria**: $29/mo in acceptable range for ≥70% of respondents; $99/seat rated acceptable (not "too expensive") by ≥50% of engineering leads/managers.

**Decision**: $29 too expensive for >30% → lower to $19/mo.

---

## Experiment 4: Distribution Channel Test

**Hypothesis**: Hacker News (Show HN) + public GitHub repo generates ≥100 waitlist signups within 72 hours, confirming organic developer community as the primary PLG channel.

**Method**: Build 1-page waitlist landing page (no-code, 1–2 hours). Post to HN as "Ask HN: Would you use a compiled programming language for AI agent workflows?" Cross-post link to LangChain Discord, Anthropic Developers Discord, AI Engineer community. Measure for 72 hours: unique visits, signups, HN upvotes, comment sentiment.

**Success criteria**: ≥100 waitlist signups; ≥50 HN upvotes; net-positive comment sentiment.

**Decision**: <100 signups + negative comments → developer community is not the right first channel. Consider enterprise outreach instead.

---

## Experiment 5: Model Quality Benchmark

**Hypothesis**: Gemma4-26B-A4B with MTP speculative decoding on AWS g5.xlarge achieves ≥85% of Claude 3.5 Sonnet quality on code tasks at ≤15% of Claude's cost per token.

**Method**: Launch g5.xlarge spot instance. Deploy vLLM:
```bash
vllm serve google/gemma-4-26B-A4B-it \
  --speculative-model google/gemma-4-26B-A4B-it-assistant \
  --num-speculative-tokens 5 --quantization awq \
  --dtype bfloat16 --max-model-len 32768
```
Run 20 code tasks (dead code detection, refactoring, test generation, documentation) through both Gemma4 and Claude 3.5 Sonnet. Blind evaluation by 3 developers from Exp 1 pool.

**Success criteria** (all four must pass):

| Metric | Threshold |
|--------|-----------|
| Quality (blind eval) | Gemma4 wins or ties in ≥40% of tasks |
| Throughput | ≥45 tok/s on A10G with MTP enabled |
| Cost | ≤$0.20 per 1M tokens |
| MTP acceptance rate | ≥60% of speculated tokens accepted |

**Total cost**: ~$105 ($10 spot + $90 evaluator gift cards + $5 Bedrock).  
**Decision**: Quality <40% win rate → Tier 2 not viable at this model size; evaluate Gemma4-A9B or drop Tier 2.
