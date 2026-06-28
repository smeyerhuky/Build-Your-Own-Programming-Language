---
type: Feature Backlog
title: "LangCraft Platform — Feature Backlog"
description: Prioritised feature backlog for the LangCraft platform. P0 = must have for launch; P1 = important; P2 = nice to have.
tags: [backlog, product, features, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Feature Backlog

*Priority: P0 = must have for MVP launch | P1 = next release | P2 = later*

---

## P0 — MVP (Q4 2026)

| ID | Feature | Description | Owner |
|---|---|---|---|
| B-001 | Cloud sandbox runtime | Containerised J0 compiler (JFlex + BYacc + JDK 17 + Unicon) running in browser | Engineering |
| B-002 | Web code editor | Monaco or CodeMirror editor with J0 syntax highlighting | Engineering |
| B-003 | Compilation pipeline viewer | Step-by-step output: token stream → parse tree → AST → TAC → VM output | Engineering |
| B-004 | Auth: email + GitHub OAuth | Secure authentication; persistent sessions | Engineering |
| B-005 | Phase 1–5 exercises | Interactive exercises for all 5 onboarding phases, ported from OKF bundle | Product |
| B-006 | Automated test harness | Compare exercise output against expected values; pass/fail feedback | Engineering |
| B-007 | Hint system | 3-level progressive hints per exercise; hint credits per day on free tier | Product |
| B-008 | LLM assistant (sandbox) | GPT-4o or Claude 3.5 integrated into sandbox; RAG over OKF content | Engineering |
| B-009 | Error explanation mode | Paste compiler error → plain-English explanation + fix suggestion | Engineering |
| B-010 | Free tier limits | 5 LLM queries/day; 60 min sandbox/day; enforced server-side | Engineering |
| B-011 | Pro tier + Stripe billing | $29/mo subscription; Stripe checkout; webhook-based activation | Engineering |
| B-012 | Academic discount flow | `.edu` email detection; 69 % discount auto-applied | Engineering |
| B-013 | Progress tracking | Per-user, per-phase progress stored in database | Engineering |
| B-014 | Responsive web app | Works on Chrome, Firefox, Safari (desktop only for v1) | Engineering |

---

## P1 — Next release (Q1–Q2 2027)

| ID | Feature | Description | Owner |
|---|---|---|---|
| B-020 | Phase 6–10 exercises | Exercises for extending J0 with new features and running the full pipeline | Product |
| B-021 | Certificate of completion | Shareable PDF certificate per phase and for full course | Product |
| B-022 | Portfolio page | Public-facing page showing completed exercises and projects | Product |
| B-023 | VS Code extension v1 | J0 syntax highlighting; "Run in LangCraft Sandbox" command | Engineering |
| B-024 | VS Code extension v2 | LSP integration: diagnostics, hover docs, go-to-definition | Engineering |
| B-025 | LangCraft CLI | `langcraft compile <file.j0>` command; installable via npm or brew | Engineering |
| B-026 | Instructor cohort tools | Create cohort; invite students; assign exercises; view progress dashboard | Product |
| B-027 | Grade export | CSV export of student progress for LMS integration | Product |
| B-028 | Enterprise SSO (SAML/OIDC) | Okta, Azure AD, Google Workspace integration | Engineering |
| B-029 | Team billing | Organisation accounts; multiple seats; admin console | Engineering |
| B-030 | Audit log | Immutable log of all compilation events per team; exportable | Engineering |
| B-031 | Showcase gallery | Public gallery of student and enterprise projects built on LangCraft | Product |
| B-032 | Discussion forum | Phase-specific community threads; Discourse integration | Product |
| B-033 | x86-64 sandbox support | Extend sandbox to support Ch 13 native code generation (requires emulation) | Engineering |

---

## P2 — Later (Q3–Q4 2027)

| ID | Feature | Description | Owner |
|---|---|---|---|
| B-040 | Language Package Registry | Publish and install J0 language extensions (new keywords, operators, types) | Engineering |
| B-041 | Multi-language support | Allow learners to build DSLs using custom grammars beyond J0 | Engineering |
| B-042 | LangCraft API | REST/WebSocket API for embedding sandboxes in external documentation | Engineering |
| B-043 | Mobile-responsive design | Full mobile support (Phase 1–3 exercises on tablet) | Engineering |
| B-044 | Localization (Spanish) | Full Spanish translation of course content and UI | Product |
| B-045 | Localization (Mandarin) | Full Mandarin translation | Product |
| B-046 | Advanced certification (exam) | LangCraft Certified Language Engineer proctored exam | Product |
| B-047 | FedRAMP authorization | Required for US government and defence enterprise customers | Engineering |
| B-048 | Data residency (EU/APAC) | Separate hosting regions; data sovereignty compliance | Engineering |
| B-049 | University partnership program | Formal curriculum partnership: LangCraft as official lab environment | Sales |
| B-050 | Private self-hosted option | Docker-based self-hosted LangCraft for air-gapped enterprise environments | Engineering |

---

## Dropped / won't do

| ID | Feature | Reason |
|---|---|---|
| B-D01 | Native mobile app (iOS/Android) | Not worth the investment at this stage; web is sufficient |
| B-D02 | In-app video lessons | Video production is expensive and not our core competency; link to YouTube instead |
| B-D03 | Full general-purpose cloud IDE | Competes with VS Code.dev; not differentiated |
| B-D04 | Support for Bison (GNU yacc) in sandbox | Scope creep; BYacc/Cup covers the course; Bison can be added in a community extension |
| B-D05 | Gamification (badges, leaderboards) | Research shows mixed results; adds complexity; defer to v2 if community asks for it |
