# HCS-10 Evolution Notes (Non-Normative)

> **Status:** Informational  
> **Scope:** Forward-looking  
> **Normativity:** None  

This document is non-normative and forward-looking. It does not propose changes to
HCS-10’s current compliance requirements, nor does it invalidate existing or
in-progress implementations. Its purpose is to capture potential evolution paths
as the HCS specification catalog scales over time.

---

## Context

HCS-10 successfully defines a rich, end-to-end interoperability model for
decentralized conversation, agent coordination, and message exchange. In doing so,
it effectively proved an important category within the HCS ecosystem.

As the number of HCS specifications grows—potentially into the hundreds or
thousands—HCS-10 also serves as a useful reference point for understanding how
different kinds of specification concerns may benefit from clearer separation and
composition.

This document treats HCS-10 not as a problem to be corrected, but as a successful
early standard whose breadth naturally exposes scaling considerations for future
specification design.

---

## Observed Pattern

HCS-10 currently encompasses multiple concerns within a single specification:

- **Message structure** (schemas and field semantics)
- **Behavioral requirements** (ordering, processing expectations, interoperability rules)
- **Use-case composition guidance** (how components are combined to build applications)

This unified approach has been effective for early adoption and clarity, but it may
not scale linearly if future specs with narrower scope converge toward similarly
large, monolithic documents.

---

## Conceptual Decomposition (Non-Normative)

One possible way to reason about future HCS specification design is to view HCS-10
as conceptually spanning three distinct layers. This framing is descriptive, not
prescriptive.

### 1. Core — Behavioral Invariants

A minimal set of normative guarantees required for interoperability, such as:

- ordering and replay expectations
- identity or signing assumptions
- processing invariants that must be shared across implementations

These invariants tend to change slowly and are strongest candidates for
behavioral-style specifications.

---

### 2. Schema Specs — Message Formats

Schema-first specifications that primarily define message structure and validation
rules, often extending a common envelope (e.g., HCS-2).

Characteristics:
- small, focused scope
- versioned independently
- minimal or no behavioral prescription
- expected to represent the majority of future HCS specs

---

### 3. Profiles — Composition / Use-Case Contracts

Composable groupings of Core + Schema specs that describe how to assemble a
particular application or domain use case.

Profiles may:
- constrain optional fields
- define required subsets
- describe interoperability expectations without introducing new protocol rules

This allows richer guidance without forcing each spec to become a full protocol.

---

## Implications for Future HCS Specs

This conceptual separation suggests an evolution path where:

- schema-first specs are encouraged by default
- behavioral specs are reserved for true interoperability requirements
- profiles capture higher-level application guidance
- a lightweight registry or index can support discoverability at scale

None of these implications require retroactive changes to HCS-10; they instead
provide a framework for future consistency as the catalog grows.

---

## Alignment With Ongoing Work

This framing reflects how newer schema-oriented HCS proposals can remain small,
composable, and focused without converging toward HCS-10-sized documents.

The author intends to align current and future PRs with this direction by:
- keeping schema-oriented specs narrowly scoped
- avoiding introduction of new envelope semantics where possible
- explicitly stating what behavior a spec does *not* prescribe

---

## Closing Note

HCS-10’s breadth is a strength—it demonstrates what is possible. These evolution
notes simply acknowledge that long-term scale may benefit from clearer layering
and composition patterns, informed directly by HCS-10’s success.
