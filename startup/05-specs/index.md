---
type: Index
title: "Conductor Labs — Specification Index"
description: Navigation index for all formal specifications governing the Conductor DSL, runtime, and REST API. Implementations must pass the conformance test suites referenced in each spec.
tags: [specs, conductor, dsl, runtime, api, conformance]
timestamp: 2026-06-28T00:00:00Z
---

# Specifications

This directory contains the three formal specifications that together define the complete contract for Conductor. Any implementation claiming Conductor compatibility must satisfy all three specifications and must pass the conformance test suites referenced within them.

---

## Specs in This Section

### [Language Specification](./language-spec.md)

Defines the `.agent` file format: lexical structure, EBNF grammar, the type system (built-in types, tool signatures, inference rules), semantic validity rules, and the template interpolation sub-language. This spec is the authoritative reference for the Conductor compiler front-end (lexer, parser, type checker). Implementations must pass `tests/conformance/language/`.

### [Runtime Specification](./runtime-spec.md)

Defines the IR instruction set and its execution semantics: the state machine for pipeline runs, the dispatcher contract for each IR opcode, context-window management and token budget accounting, tool executor contracts, idempotency guarantees, and multi-tenant isolation rules. This spec is the authoritative reference for the IR walker and tool executor. Implementations must pass `tests/conformance/runtime/`.

### [API Specification](./api-spec.md)

Defines the REST and Server-Sent Events surface at `https://api.conductorlabs.io/v1`: authentication scheme, rate-limit tiers, the `/compile` and `/runs` family of endpoints, pipeline management endpoints, the SSE event stream, the OpenAI-compatible inference proxy, and the unified error response format. This spec is the authoritative reference for the Conductor HTTP API layer. Implementations must pass `tests/conformance/api/`.

---

## Conformance Requirement

A Conductor implementation is conformant when and only when:

1. The compiler front-end accepts all programs in `tests/conformance/language/valid/` and rejects all programs in `tests/conformance/language/invalid/` with the exact error codes enumerated in the Language Specification §4.4.
2. The runtime executes all IR fixtures in `tests/conformance/runtime/` and produces outputs matching the golden files within the documented tolerance for non-deterministic LLM responses.
3. The HTTP layer responds to all requests in `tests/conformance/api/` with the status codes and JSON schemas defined in the API Specification.

Partial conformance (e.g., language-only for a linter tool) must be declared explicitly in the implementation's `conductor.json` manifest under the `"conformance"` key.

---

## Related Resources

- [harness/README.md](../../harness/README.md) — Architecture North Star; reliability mathematics and harness engineering rationale
- [wiki/](../../wiki/) — OKF wiki with compiler theory cross-references (symbol tables, type inference, IR design)
- [startup/](../) — Full startup documentation index
