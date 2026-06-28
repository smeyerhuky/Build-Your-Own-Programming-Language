---
type: Validation
title: "Go/No-Go Criteria by Phase — Conductor Labs"
description: Numeric, measurable go/no-go gates for every Conductor development phase — Pre-Build, Phase 1, Phase 2, Phase 3 — plus kill criteria and a technical risk register with probability, impact, mitigation, and escalation triggers.
resource: /startup/04-validation/go-no-go-criteria.md
tags: [validation, go-no-go, metrics, phases, risk]
timestamp: 2026-06-28T00:00:00Z
---

# Go/No-Go Criteria by Phase

Every criterion below has a binary outcome: pass or fail. There is no "mostly pass." At the end of each phase, every item in the gate is evaluated. A single NO-GO trigger failing is sufficient to halt advancement to the next phase until resolved or formally waived with a documented rationale.

---

## Pre-Build Gate

**Purpose**: Validate that the customer pain is real and that enough people want to pay for a solution before any Phase 1 code is written.

**Timeline**: 2 weeks (experiments run concurrently; see [experiments.md](experiments.md)).

**Owner**: Founder.

### Criteria

- [ ] **INT-01** — 10 developer interviews completed (targeting active LangChain/LangGraph users).
- [ ] **INT-02** — ≥7 of 10 interview subjects unprompted surface at least one of these three pain points: (a) lack of type safety in tool contracts, (b) difficulty switching LLM providers, (c) inability to audit or replay execution traces.
- [ ] **INT-03** — ≥5 of 10 interview subjects say they would pay $29/mo for a working solution.
- [ ] **INT-04** — ≥3 interview subjects agree to be contacted when the Phase 1 beta ships.
- [ ] **SYN-01** — Syntax usability test: ≥4 of 5 developers produce a syntactically valid `.agent` pipeline in under 10 minutes without documentation.
- [ ] **PRC-01** — Pricing survey: $29/mo falls within the "acceptable" price band for ≥70% of survey respondents.
- [ ] **DST-01** — Distribution channel test: ≥100 waitlist signups within 72 hours of the HN post.

### NO-GO Triggers

| Trigger | Threshold |
|---------|-----------|
| Pain not confirmed | <5 of 10 interviews surface any of the three core pain points |
| No willingness to pay | 0 of 10 interview subjects say they'd pay $29/mo |
| No beta commitments | 0 people agree to be beta users |
| Syntax unusable | <3 of 5 developers complete the syntax test in 10 minutes |

---

## Phase 1 Exit Gate

**Purpose**: Verify the MVP is technically sound and that real users are running real pipelines.

### Technical Criteria

- [ ] **PERF-01** — Compile latency: p50 <200ms, p99 <1,000ms on the Phase 1 test suite.
- [ ] **TYPE-01** — Type checker true positive rate: ≥90% on 20 intentionally type-incorrect `.agent` files.
- [ ] **TYPE-02** — Type checker false positive rate: 0 false positives on 20 correct pipelines.
- [ ] **E2E-01** — The canonical `refactor_session.agent` compiles and executes end-to-end with real Bedrock.
- [ ] **ISO-01** — Multi-tenant isolation: 10 cross-tenant access attempts all return 403 or empty result.
- [ ] **BILL-01** — Stripe usage records match actual Bedrock token counts to within ±5%.
- [ ] **UPTIME-01** — Control plane API uptime ≥99% over the 2-week beta period.
- [ ] **SANDBOX-01** — 5 shell sandbox escape attempt scripts all fail to escape.

### Business Criteria

- [ ] **USR-01** — 10 distinct users executed at least one real pipeline during the beta period.
- [ ] **PAY-01** — ≥3 users paying $29/mo Builder tier at end of Month 2.
- [ ] **NPS-01** — NPS from beta user survey: ≥40.
- [ ] **RET-01** — At least 1 paying user has stated they replaced LangChain code with a `.agent` pipeline.

### NO-GO Triggers for Phase 2

| Trigger | Threshold |
|---------|-----------|
| No paying users | <3 paying Builder subscribers at end of Month 2 |
| Compile latency | p99 >2,000ms on the test suite |
| Multi-tenant isolation failure | Any confirmed cross-tenant data access |

---

## Phase 2 Exit Gate

### Technical Criteria

- [ ] **EXT-01** — VS Code extension: ≥50 installs, <10 open bug issues.
- [ ] **MCP-01** — MCP compliance: passes all 10 test cases in the official MCP spec test suite.
- [ ] **CI-01** — GitHub Actions integration completes in under 5 minutes on a standard runner.
- [ ] **TMPL-01** — 5 template pipelines runnable by new users without modification.

### Business Criteria

- [ ] **MAU-01** — 100 Monthly Active Users.
- [ ] **PAY-02** — 20 active paying Builder subscribers at end of Month 4.
- [ ] **HN-01** — Show HN achieves ≥100 upvotes and ≥50 comments.
- [ ] **GH-01** — ≥500 GitHub stars at end of Month 4.

---

## Phase 3 Exit Gate

### Technical Criteria

- [ ] **GPU-01** — Gemma4-26B-A4B: p50 TTFT <500ms, p99 <2,000ms, in all 3 deployed regions.
- [ ] **GPU-02** — vLLM uptime ≥99.5% across all regions over 30-day window.
- [ ] **GPU-03** — MTP speculative decoding acceptance rate ≥65% on coding-task prompts.
- [ ] **SSO-01** — Okta SAML2 integration passes conformance test suite.
- [ ] **MKT-01** — AWS Marketplace listing live and purchasable.

### Business Criteria

- [ ] **ENT-01** — 3 enterprise LOIs signed for ≥$2,000/mo each.
- [ ] **MAU-02** — 200 MAU total at end of Month 6.
- [ ] **PAY-03** — 50 active paying subscribers combined.

---

## Kill Criteria

| Condition | When | Why |
|-----------|------|-----|
| **Zero retention** | After Phase 1 | 0 users run a second pipeline. No value demonstrated. |
| **Sustained negative NPS** | After Phase 2 | NPS consistently <0 across 2 consecutive monthly cohorts. |
| **Competitor platform launch** | At any time | AI lab ships a compiled DSL for agents with free tier. |
| **Security incident** | At any time | Confirmed cross-tenant breach or sandbox escape. |

---

## Technical Risk Register

| Risk | Probability | Impact | Mitigation | Escalation Trigger |
|------|-------------|--------|------------|-------------------|
| Type checker false negatives | Medium | High | 20-case negative test suite; property-based fuzzing | >5% false negative rate |
| Bedrock API rate limiting | Medium | Medium | Per-tenant limits; exponential backoff; quota increase requests | >1% runs fail due to ThrottlingException |
| vLLM spot interruption | High | Medium | Checkpoint + resume protocol; ASG replacement | >0 confirmed run data losses |
| Shell sandbox escape | Low | Critical | Docker seccomp; non-root; read-only rootfs; pen test | Any confirmed escape |
| LLM output JSON parse failure | High | Medium | Retry with temp=0; structured output API in Phase 2 | >10% parse failures in any 1-hour window |
| Stripe webhook failures | Low | High | Idempotent handler; daily reconciliation job | >0.1% LLM calls with no Stripe record after 24h |
| Postgres RLS bypass | Low | Critical | Middleware enforces tenant_id; RLS integration tests | Any cross-tenant data in integration tests |
