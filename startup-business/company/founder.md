---
type: Founder Profile
title: "Clinton L. Jeffery — Founder Profile"
description: Biography, academic background, research history, and founding motivation of Clinton L. Jeffery, the creator of J0 and the Build Your Own Programming Language course.
tags: [founder, company, clinton-jeffery, langcraft, startup-business]
timestamp: 2026-06-28T00:00:00Z
---

# Clinton L. Jeffery — Founder

## At a glance

| Field | Detail |
|---|---|
| Name | Clinton L. Jeffery |
| Title | Professor and Chair, Department of Computer Science and Engineering |
| Institution | New Mexico Institute of Mining and Technology (New Mexico Tech) |
| Location | Socorro, New Mexico, USA |
| Education | B.S. Computer Science, University of Washington; M.S. and Ph.D. Computer Science, University of Arizona |
| Languages invented | Unicon (with co-inventors); J0 (solo, as a teaching language) |
| Key publication | *Build Your Own Programming Language*, Packt Publishing, 2021 (ISBN 978-1-80020-480-5) |
| GitHub | github.com/jeffery-nm (course repository) |

---

## Academic background

Clinton received his undergraduate degree from the **University of Washington**, where he developed an early interest in programming language theory and systems software. He went on to complete both his Master's and Ph.D. at the **University of Arizona**, focusing on program monitoring, debugging, and language design.

After completing his doctorate he joined the faculty at **New Mexico Tech**, where he is currently **Professor and Chair of the Department of Computer Science and Engineering**. His academic career spans more than two decades of teaching, research, and language design.

---

## Research contributions

### Unicon programming language

Clinton is a co-inventor of the **Unicon programming language**, a superset of the Icon programming language. Unicon extends Icon with:

- **Object-oriented programming** — classes, inheritance, and operator overloading.
- **Concurrent programming** — threads, mutexes, and condition variables.
- **Database access** — ODBC bindings and ISAM file formats.
- **Network programming** — streams, datagrams, and internet I/O.
- **2D/3D graphics** — a portable OpenGL-based graphics library.

Unicon is hosted at [unicon.org](https://unicon.org) and is used in production for scientific computing and educational applications. It remains an active open-source project with contributions from researchers worldwide.

### Program monitoring and debugging

Clinton has published extensively on tools for understanding and debugging programs, including work on **program visualisation**, **execution monitoring**, and **dynamic analysis**. This work directly informs the diagnostics and tooling features of the LangCraft platform.

### Compiler and language education

Over the course of his career Clinton has refined the pedagogy of teaching compiler construction. His approach — building a real, working compiler for a realistic language subset (J0) using off-the-shelf tools (JFlex, BYacc, Cup) — has been validated with hundreds of students across multiple universities.

---

## The J0 language

Clinton designed **J0** specifically for the *Build Your Own Programming Language* course. J0 is a subset of Java that is:

- **Small enough** to implement in a single semester course (no generics, no concurrency, no reflection).
- **Real enough** to teach every major phase of a compiler: lexing, parsing, AST construction, symbol tables, type checking, intermediate code generation, interpretation, and native code generation.
- **Extensible** by design — each chapter adds features to the language, teaching students how language evolution works.

J0 is the core technology asset of LangCraft and is documented formally in [RFC-001](../rfc/rfc-001-j0-language-spec.md).

---

## Founding motivation

Clinton's motivation for starting LangCraft arose from three observations made over years of teaching:

1. **The book alone is not enough.** Students and self-learners frequently get stuck on environment setup, toolchain bugs, and the gap between reading about a concept and implementing it. The course needs tooling, scaffolding, and instant feedback.

2. **Enterprises have the same problem.** Companies building DSLs for configuration, workflow, query, or policy languages face exactly the challenges the book addresses — but they have no accessible, production-quality toolchain. They either reinvent everything from scratch or adopt heavyweight frameworks with steep learning curves.

3. **LLMs change the equation.** The emergence of capable code-generation models means that a well-structured, LLM-friendly curriculum (OKF-formatted, with concept pages, prompt templates, and error catalogues) can dramatically accelerate learning. LangCraft is positioned to be the best LLM-integrated compiler construction platform available.

> "I've spent twenty years teaching students to build programming languages. The tools exist. The knowledge exists. What's missing is the platform that makes it accessible to anyone who needs it." — Clinton L. Jeffery

---

## Publications (selected)

| Year | Title | Venue |
|---|---|---|
| 2021 | *Build Your Own Programming Language* | Packt Publishing |
| 2018 | *Programming with Unicon* (2nd ed.) | Online free book, unicon.org |
| 2015 | *The Implementation of Icon and Unicon* | Online free book, unicon.org |
| 2002 | *Graphics Programming in Icon* | Peer-to-Peer Communications |
| 1994 | "Program Monitoring and Visualization" | Ph.D. Dissertation, University of Arizona |

---

## Contact

- **University**: New Mexico Institute of Mining and Technology, Socorro NM 87801
- **Startup**: LangCraft Inc. (operational address TBD upon incorporation)
- **Email**: cjeffery@nmt.edu (academic); clinton@langcraft.io (startup, pending domain)
