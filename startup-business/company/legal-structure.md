---
type: Legal Structure
title: "LangCraft — Legal Structure"
description: Delaware C-Corp structure, IP ownership, licensing model, equity table, and key legal considerations for LangCraft Inc.
tags: [legal, company, equity, ip, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Legal Structure

*This document is for internal planning purposes. It is not legal advice. Engage qualified legal counsel before incorporation.*

---

## Corporate structure

| Field | Detail |
|---|---|
| Entity type | Delaware C-Corporation |
| Registered agent | To be determined (National Registered Agents, Inc. or equivalent) |
| State of incorporation | Delaware |
| Principal office | Socorro, New Mexico, USA |
| Fiscal year end | December 31 |
| Target incorporation date | Q3 2026 |

**Why Delaware C-Corp?** Standard choice for venture-backed U.S. startups. Institutional investors expect it. Delaware's Court of Chancery provides predictable corporate governance case law.

---

## Intellectual property

### What LangCraft owns

| Asset | Status | Notes |
|---|---|---|
| J0 compiler source code (Unicon version) | To be assigned | Currently authored by Clinton; IP assignment agreement required at incorporation |
| J0 compiler source code (Java version) | To be assigned | Same as above |
| `javalex.l` lexer specification | To be assigned | Core lexer file used in all chapters |
| `j0gram.y` grammar specification | To be assigned | Core parser file used in all chapters |
| OKF wiki content | To be assigned | All `.md` files in `/wiki/` and `/startup/` |
| `startup-business/` bundle (this folder) | To be assigned | Business documentation |
| `langcraft.io` domain | To be registered | Domain purchase pending |

### What LangCraft does NOT own

| Asset | Owner | Notes |
|---|---|---|
| *Build Your Own Programming Language* book text | Packt Publishing | Copyright held by Packt; LangCraft has no ownership claim over book text |
| Unicon programming language | Unicon Foundation / co-inventors | LangCraft can use Unicon (open source) but does not own it |
| JFlex | JFlex project (open source, BSD) | Freely usable; no ownership |
| BYacc / Cup | Various open-source projects | Freely usable; no ownership |

### IP assignment

At incorporation, Clinton L. Jeffery will execute an **IP Assignment Agreement** transferring ownership of all startup-relevant IP (J0 compiler source, OKF content, domain registrations) to LangCraft Inc. A carve-out will be negotiated for purely academic/research uses and for the Unicon language project.

---

## Licensing model

LangCraft operates a **dual-license model**:

### Open core (MIT License)
The following assets will be released and maintained under the MIT License:
- J0 compiler source code (all language phases, both Java and Unicon implementations)
- OKF wiki documentation
- Technical onboarding bundle (`/startup/`)
- JFlex lexer and BYacc grammar specifications

**Why MIT?** Permissive licensing maximises adoption, academic use, and community contributions. It is consistent with the existing repository licensing (see [LICENSE](../../LICENSE)).

### Commercial license
The following assets will require a commercial license:
- LangCraft Cloud platform (sandbox runtime, CI/CD for languages, hosted deployments)
- Enterprise IDE plugin (VS Code extension with advanced LSP features)
- LangCraft Enterprise (SSO, audit logs, SLA, multi-tenant management console)
- Language Package Registry (private package hosting)

---

## Equity structure (target at incorporation)

*Pre-seed target. Subject to negotiation.*

| Stakeholder | Shares | % (pre-seed) | Vest |
|---|---|---|---|
| Clinton L. Jeffery (CTO/Co-founder) | 4 000 000 | 40 % | 4-yr, 1-yr cliff |
| CEO Co-founder (open) | 3 500 000 | 35 % | 4-yr, 1-yr cliff |
| Employee option pool | 1 500 000 | 15 % | Per-grant schedule |
| Advisors (4 × 0.25 %) | 250 000 | 2.5 % | 2-yr, 6-mo cliff |
| Pre-seed investors | 750 000 | 7.5 % | N/A (purchased) |
| **Total** | **10 000 000** | **100 %** | |

*Post-seed, the option pool will expand to 20 % of total shares outstanding. Pre-seed investor shares are priced at a valuation cap TBD.*

---

## Key legal to-dos before launch

- [ ] Engage startup attorney (recommend Cooley, Wilson Sonsini, or Gunderson Dettmer)
- [ ] File Delaware C-Corp incorporation
- [ ] Execute founders' IP assignment agreements
- [ ] Execute 83(b) elections within 30 days of share issuance
- [ ] Open corporate bank account (Mercury or Silicon Valley Bank)
- [ ] Register `langcraft.io` domain
- [ ] Review Packt Publishing agreement for any non-compete or exclusivity clauses
- [ ] File trademark application for "LangCraft" (USPTO Class 42: Software as a Service)
- [ ] Draft advisor agreements
- [ ] Set up cap table management (Carta)
