---
title: HCS-27 - Skill Trust Scores and Adapters
description: A deterministic trust-scoring profile for HCS-26 skill releases, including adapter interface, aggregation rules, and baseline adapter set.
sidebar_position: 27
---

# HCS-27 Standard: Skill Trust Scores and Adapters

### Status: Draft

### Version: 0.1

### Table of Contents

- [Authors](#authors)
  - [Primary Author](#primary-author)
  - [Additional Authors](#additional-authors)
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Interpretation Guidance (Informative)](#interpretation-guidance-informative)
- [Relationship to HCS-25 and HCS-26](#relationship-to-hcs-25-and-hcs-26)
- [Terminology](#terminology)
- [Specification](#specification)
  - [Subject Model](#subject-model)
  - [Adapter Contract](#adapter-contract)
  - [Contribution Modes](#contribution-modes)
  - [Execution Profiles](#execution-profiles)
  - [Normalization and Aggregation](#normalization-and-aggregation)
  - [Initial Baseline Adapter Set](#initial-baseline-adapter-set)
    - [1) Upvotes Adapter](#1-upvotes-adapter)
    - [2) Verified Adapter](#2-verified-adapter)
    - [3) Metadata Completeness Adapter](#3-metadata-completeness-adapter)
    - [4) Safety Adapter](#4-safety-adapter)
    - [5) GitHub Repository Adapter](#5-github-repository-adapter)
  - [Output Schema](#output-schema)
  - [Versioning](#versioning)
- [Security Considerations](#security-considerations)
- [Privacy Considerations](#privacy-considerations)
- [Conformance](#conformance)
- [References](#references)
- [License](#license)

## Authors

### Primary Author

- Michael Kantor [https://twitter.com/kantorcodes](https://twitter.com/kantorcodes)

### Additional Authors

None.

## Abstract

HCS-27 defines a deterministic trust-scoring profile for **HCS-26 skill releases**. It standardizes:

1. the trust adapter contract for skills;
2. applicability and contribution semantics;
3. normalization and weighted aggregation rules; and
4. an initial adapter baseline for production interoperability.

The result is a reproducible **skill trust score** and explainable per-adapter component breakdown in the range **0–100**.

## Motivation

HCS-26 enables decentralized publication and discovery of skills. Consumers and integrators still need an objective way to compare releases across:

- provenance and verification evidence;
- package safety and operational risk;
- community usage signals; and
- metadata quality and maintainability.

HCS-27 provides a portable scoring profile that can be used consistently across registries, marketplaces, and clients without coupling to one backend implementation.

## Interpretation Guidance (Informative)

HCS-27 defines a **methodology for deriving scores**, not canonical reputation identity.

Implementers and consumers SHOULD treat HCS-27 outputs as:

- **Context-bound** (subject + execution profile + adapter configuration version),
- **Derived and versioned** (signal freshness and adapter logic matter), and
- **Decision support** (one input among policy, provenance, and human review).

Implementations MUST NOT use HCS-27 totals as the sole authoritative basis for irreversible exclusion or enforcement decisions.

## Relationship to HCS-25 and HCS-26

- **HCS-25** defines a general trust-score methodology.
- **HCS-27** defines a concrete, interoperable profile of that methodology for skills.
- **HCS-26** defines the skill publication model and release artifacts.

In short: HCS-26 defines skill releases, HCS-25 defines generic scoring principles, and HCS-27 binds those principles to skill-specific adapters and outputs.

## Terminology

- **Skill Subject**: An HCS-26 skill release identified by `network + name + version`.
- **Trust Adapter**: Deterministic function mapping skill data/signals to normalized components.
- **Component**: A normalized value in `[0,100]` emitted by one adapter.
- **Adapter Total**: Mean of components emitted by one adapter.
- **Composite Skill Trust Score**: Weighted mean of adapter totals in `[0,100]`.
- **Execution Profile**: Runtime mode deciding which adapters are eligible (for example, local-only vs external I/O enabled).

Normative language such as **MUST**, **SHOULD**, and **MAY** follows RFC 2119.

## Specification

### Subject Model

A conforming implementation MUST compute trust for one skill release record containing:

- `network`, `name`, `version`,
- release metadata (`description`, `tags`, `category`, `homepage`, `repo`, `commit`),
- published files (including file names and hashes),
- verification fields (`verified`, optional verification signals),
- safety fields (optional persisted safety summary / findings),
- voting context (`upvotes`).

### Adapter Contract

A skill trust adapter MUST expose:

- `id`: stable adapter identifier;
- `weight` (optional; default `1`);
- `contributionMode` (optional; default `conditional`);
- `defaultComponentKey` and/or `componentKeys`;
- `appliesTo(subject, context) -> boolean`;
- `inferExternalId(subject)` (optional); and
- `fetchScores(subject, externalId, context) -> map<string, number> | null`.

Returned scores MUST be finite numeric values. Non-finite values MUST be ignored.

### Contribution Modes

The adapter contribution mode controls denominator behavior:

- `conditional` (default): adapter participates in denominator only if it emitted scores.
- `universal`: if applicable, adapter participates in denominator even when it emits no scores.
- `scoped`: same denominator behavior as universal; intended for profile-scoped policies.

### Execution Profiles

A conforming implementation SHOULD support at least:

1. **Read profile** (`includeExternal=false`): external lookups disabled for low-latency responses.
2. **Refresh profile** (`includeExternal=true`): external lookups allowed for background trust refresh.

The same scoring/aggregation algorithm MUST apply to both profiles; only adapter applicability/input freshness may differ.

### Normalization and Aggregation

Given raw adapter component map `M`:

1. Clamp each component to `[0,100]`.
2. Group components by adapter id prefix (`adapter.component`).
3. For configured denominator adapters, missing declared components MUST be materialized as `0`.
4. Compute each adapter total as arithmetic mean of that adapter’s component values.
5. Compute composite total as weighted mean of adapter totals:

```
total = round2( sum( weight_i * adapterTotal_i ) / sum(weight_i) )
```

If the denominator is empty, total MUST be `0`.

### Initial Baseline Adapter Set

This baseline is the interoperable minimum for HCS-27 implementations.

#### 1) Upvotes Adapter

- `id`: `upvotes`
- `weight`: `1`
- Components: `upvotes.score`
- Applies in all profiles.
- Score function:

```
score = clamp( round( 100 * (1 - e^(-upvotes / 20)) ), 0, 100 )
```

#### 2) Verified Adapter

- `id`: `verified`
- `weight`: `1`
- Components:
  - `verified.score`
  - `verified.publisherBound`
  - `verified.repoCommitIntegrity`
  - `verified.manifestIntegrity`
  - `verified.domainProof`
- Applies in all profiles.

Signal weights for `verified.score`:

- `publisherBound = 20`
- `repoCommitIntegrity = 40`
- `manifestIntegrity = 30`
- `domainProof = 10`

If subject is not verified, `verified.score` MUST be `0`, while component flags MAY still expose available partial evidence as `100|0`.

If subject is verified and no verification signal object exists, implementation MAY return all verified components as `100` for backward compatibility.

#### 3) Metadata Completeness Adapter

- `id`: `metadata`
- `weight`: `0.75`
- Components:
  - `metadata.score`
  - `metadata.links`
  - `metadata.description`
  - `metadata.taxonomy`
  - `metadata.provenance`
- Applies in all profiles.

Component scoring:

- **links**: `100` when both homepage and repo are present, `60` when either is present, else `0`.
- **description**: length thresholds (`>=160 => 100`, `>=80 => 85`, `>=30 => 65`, `>=10 => 40`, else `0`).
- **taxonomy**: based on category presence and non-empty tag count.
- **provenance**: `100` for repo+commit, `70` repo only, `40` commit only, else `0`.

Aggregate:

```
metadata.score = round2(
  clamp(
    links*0.30 + description*0.25 + taxonomy*0.20 + provenance*0.25,
    0,
    100
  )
)
```

#### 4) Safety Adapter

- `id`: `safety`
- `weight`: `1`
- `contributionMode`: `universal`
- Components: `safety.score`
- Applies in all profiles.

Behavior:

- In read profile, implementation SHOULD use persisted or fallback score.
- In refresh profile, implementation MAY execute an external safety scan.

For Cisco-scan compatible flows, scanner output SHOULD be normalized to `[0,100]` and persisted for read-time reuse.

#### 5) GitHub Repository Adapter

- `id`: `github`
- `weight`: `1`
- Components:
  - `github.score`
  - `github.repository`
  - `github.community`
  - `github.maintenance`
  - `github.resilience`
- Applies only when external lookups are enabled and a valid GitHub repo URL exists.

Signals are derived from GitHub repository metadata (stars, watchers/subscribers, open issues, created timestamp, pushed timestamp) and anti-abuse penalties.

The final GitHub score MUST be clamped to `[0,100]`. Implementations SHOULD cache results and apply short-term backoff on upstream rate limiting.

### Output Schema

A conforming response MUST include:

- `trustScores.total` in `[0,100]`, and
- optional per-component entries keyed as `<adapterId>.<componentKey>`.

Example:

```json
{
  "trustScores": {
    "upvotes.score": 28.19,
    "verified.score": 90,
    "verified.publisherBound": 100,
    "verified.repoCommitIntegrity": 100,
    "verified.manifestIntegrity": 100,
    "verified.domainProof": 0,
    "metadata.score": 86.5,
    "safety.score": 94,
    "github.score": 72.34,
    "total": 74.88
  }
}
```

### Versioning

Implementations MUST version:

- adapter set membership,
- adapter weights,
- normalization rules, and
- external signal interpretation.

Any change to these MUST be treated as a scoring configuration version change.

## Security Considerations

- External adapters SHOULD enforce bounded timeouts and rate limits.
- Implementations SHOULD isolate external scanners and untrusted package parsing.
- Trust outputs SHOULD include enough component granularity for explainability and anomaly review.
- Backoff and caching SHOULD be used to reduce upstream dependency abuse.

## Privacy Considerations

- Public trust outputs SHOULD avoid exposing sensitive internal operational data.
- Identifiers used for scoring SHOULD be limited to public skill metadata and public signal sources unless explicit consent exists.

## Conformance

An implementation conforms to HCS-27 if it:

1. Implements the adapter contract and contribution semantics in this document.
2. Produces clamped `[0,100]` component values and weighted composite totals.
3. Supports the baseline adapter set defined above (or clearly declares omitted adapters and resulting profile variance).
4. Exposes an explainable component breakdown with the total score.

## References

- HCS-25: AI Trust Score Methodology (`./hcs-25.md`)
- HCS-26: Decentralized Agent Skills Registry (`./hcs-26.md`)
- RFC 2119 (`https://www.rfc-editor.org/rfc/rfc2119`)

## License

Apache-2.0
