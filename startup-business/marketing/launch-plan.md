---
type: Launch Plan
title: "LangCraft — 90-Day Launch Plan"
description: Week-by-week execution plan for LangCraft's public launch covering pre-launch (weeks 1-8), launch week, and post-launch growth loop (weeks 9-13).
tags: [launch, marketing, execution, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# 90-Day Launch Plan

---

## Overview

| Phase | Weeks | Goal |
|---|---|---|
| Pre-launch | Weeks 1–8 | Build the product; grow waitlist to 500; prepare all launch assets |
| Launch week | Week 9 | Public launch; Hacker News; 1 000 signups in 7 days |
| Post-launch growth | Weeks 10–13 | Sustain growth; first 100 Pro conversions; iterate on feedback |

---

## Pre-launch — Weeks 1–8

### Week 1–2 — Foundation

**Product:**
- [ ] Cloud sandbox MVP running (all 5 phases, automated tests)
- [ ] Auth (email + GitHub OAuth) working
- [ ] Free tier limits enforced

**Marketing:**
- [ ] Landing page live at `langcraft.io` with waitlist email capture
- [ ] Announcement tweet from Clinton: "I'm building something new — LangCraft. Interactive compiler construction platform. Waitlist open." Link to `langcraft.io`.
- [ ] Update GitHub README: add "LangCraft waitlist →" CTA above the fold.
- [ ] Register `langcraft.io` domain; set up email `clinton@langcraft.io`.

**Target:** 100 waitlist signups.

---

### Week 3–4 — Content seeding

**Content:**
- [ ] Publish Blog Post 1: "How I turned a compiler textbook into a startup"
- [ ] Post to Hacker News (not the launch post — a genuine "I wrote this" share of the blog post)
- [ ] Share to r/ProgrammingLanguages and r/Compilers with context, not as promotion

**Product:**
- [ ] LLM assistant integrated (GPT-4o or Claude 3.5)
- [ ] Beta access granted to first 50 waitlist subscribers

**Target:** 250 waitlist signups; beta user NPS measurement begins.

---

### Week 5–6 — Beta refinement

**Product:**
- [ ] Fix top 10 issues from beta user feedback
- [ ] TTFB measurement: instrument and verify <10 min for 80 % of new users
- [ ] Pro tier billing working end-to-end (Stripe)

**Content:**
- [ ] Publish Blog Post 2: "The J0 compiler pipeline in 10 diagrams"
- [ ] Record YouTube video 1: "Build your first lexer in LangCraft (live demo)"
- [ ] Draft the Hacker News "Show HN" post — have it reviewed by 3 people

**Target:** 400 waitlist signups; 3 beta Pro conversions (validates billing).

---

### Week 7–8 — Launch preparation

**Product:**
- [ ] All Phase 1–5 exercises verified and polished
- [ ] Error messages reviewed — every error links to an OKF concept page
- [ ] Mobile-responsive check (Phase 1 must work on tablet)

**Marketing:**
- [ ] Write announcement email to waitlist (day-before-launch)
- [ ] Prepare social posts for launch day (Twitter/X, LinkedIn)
- [ ] Reach out to 5 developer newsletter editors with an exclusive preview
- [ ] Identify the 3 best "Show HN" timing windows (Tuesday 9am ET is historically strong)
- [ ] Prepare Clinton for media: short bio, headshot, 3-sentence product description

**Legal:**
- [ ] Terms of service and privacy policy live on `langcraft.io`
- [ ] GDPR cookie consent implemented

**Target:** 500 waitlist signups; product fully ready; all launch assets prepared.

---

## Launch Week — Week 9

### Day 1 (Monday) — Soft launch

- [ ] Send waitlist email: "LangCraft is live — you're first in."
- [ ] Post to Reddit (r/ProgrammingLanguages, r/Compilers, r/learnprogramming)
- [ ] Tweet: "LangCraft is live. Build a real compiler in your browser — no installation. [link]"
- [ ] Post to LinkedIn: professional framing for instructors and enterprise audience
- [ ] Submit to Product Hunt (schedule for Wednesday to avoid competing with Monday HN)

**Target:** 200 signups on Day 1.

---

### Day 2 (Tuesday) — Hacker News

- [ ] Post "Show HN: LangCraft — Interactive compiler construction in the browser" at 9am ET
- [ ] Clinton monitors and responds to every comment within 30 minutes for first 4 hours
- [ ] Do not delete or edit the post
- [ ] Engage with every substantive technical question

**What makes a good Show HN:**
- Authentic voice: "I wrote a book about this in 2021. Now I built a platform."
- Specific and honest: "Takes under 10 minutes to get from signup to a running J0 program."
- Invites technical engagement: "Happy to talk about the lexer/parser architecture."

**Target:** Front page for 6+ hours; 200+ points; 500 signups from HN.

---

### Day 3 (Wednesday) — Product Hunt

- [ ] Product Hunt listing goes live: "LangCraft — Build a compiler in your browser"
- [ ] Ask early users to upvote (in the waitlist email: "If you like what we built, upvote us on Product Hunt")
- [ ] Respond to all PH comments

---

### Day 4–7 — Momentum

- [ ] Publish Blog Post 3 (pre-written): "What I learned building a compiler construction platform"
- [ ] Share YouTube video from week 3-4 as "follow-up resource" in HN comments
- [ ] Monitor signups, Phase 1 completion rate, LLM assistant satisfaction
- [ ] Fix any critical bugs immediately (24-hour fix SLA for P0 bugs during launch week)

**Week 9 target:** 1 000 signups; 25 Pro conversions; NPS >50.

---

## Post-Launch Growth — Weeks 10–13

### Week 10–11 — Iteration

- [ ] Survey all Phase 1 completers: "What was the hardest part? What would you improve?"
- [ ] Identify top 3 friction points from analytics; fix in sprint
- [ ] Publish Blog Post 4: "Adding a new keyword to J0 in 15 minutes" (tutorial with sandbox link)
- [ ] YouTube Video 2: "From source code to token stream — the J0 lexer"

### Week 12–13 — First Pro push

- [ ] Send "upgrade" email to all users who completed Phase 3 but have not upgraded
- [ ] Test two upgrade prompts: (a) sandbox time exhaustion, (b) Phase 3 completion modal
- [ ] Reach out to university instructors identified from academic email domains in signup list
- [ ] Identify first enterprise lead from company email signups

**Month 3 target:** 2 000 registered users; 100 Pro subscribers ($2 900 MRR); TTFB <8 min for 85 % of users.

---

## Success criteria (end of day 90)

| Metric | Target |
|---|---|
| Registered users | 2 500+ |
| Pro subscribers | 100+ |
| MRR | $2 900+ |
| Phase 1 completion rate | >60 % of signups |
| NPS | >50 |
| Hacker News Show HN | Front page; 200+ points |
| TTFB | <10 min for 80 % of new users |
| Blog organic sessions/month | 1 000+ |
