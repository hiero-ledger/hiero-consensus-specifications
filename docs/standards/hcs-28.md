---
title: HCS-28 - Skill Trust Scores and Adapters
description: A deterministic trust-scoring standard for HCS-26 skill releases, including independent profiles, adapter interfaces, and aggregation rules.
sidebar_position: 28
---

# HCS-28 Standard: Skill Trust Scores and Adapters

### Status: Draft

### Version: 0.1

### Table of Contents

- [Authors](#authors)
  - [Primary Author](#primary-author)
  - [Additional Authors](#additional-authors)
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Interpretation Guidance (Informative)](#interpretation-guidance-informative)
- [Relationship to HCS-26](#relationship-to-hcs-26)
- [Terminology](#terminology)
- [Specification](#specification)
  - [Subject Model](#subject-model)
  - [Baseline Signal Semantics](#baseline-signal-semantics)
  - [Adapter Contract](#adapter-contract)
  - [Contribution Modes](#contribution-modes)
  - [Execution Profiles](#execution-profiles)
  - [Normalization and Aggregation](#normalization-and-aggregation)
  - [Baseline Adapter Catalog](#baseline-adapter-catalog)
    - [Verification Adapters](#verification-adapters)
    - [Metadata Adapters](#metadata-adapters)
    - [Upvotes Adapter](#upvotes-adapter)
    - [Safety Adapter](#safety-adapter)
    - [Repository Health Adapter](#repository-health-adapter)
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

HCS-28 defines a deterministic trust-scoring standard for **HCS-26 skill releases**. It standardizes:

1. the trust adapter contract for skills;
2. applicability and contribution semantics;
3. normalization and weighted aggregation rules; and
4. an initial baseline adapter set for interoperability.

The result is a reproducible **skill trust score** and explainable per-adapter component breakdown in the range **0–100**.

## Motivation

HCS-26 enables decentralized publication and discovery of skills. Consumers and integrators still need an objective way to compare releases across:

- provenance and verification evidence;
- package safety and operational risk;
- community usage signals; and
- metadata quality and maintainability.

HCS-28 provides a portable scoring standard that can be used consistently across registries, marketplaces, and clients without coupling to one backend implementation.

## Interpretation Guidance (Informative)

HCS-28 defines a **methodology for deriving scores**, not canonical reputation identity.

Implementers and consumers SHOULD treat HCS-28 outputs as:

- **Context-bound** (subject + execution profile + adapter configuration version),
- **Derived and versioned** (signal freshness and adapter logic matter), and
- **Decision support** (one input among policy, provenance, and human review).

Implementations MUST NOT use HCS-28 totals as the sole authoritative basis for irreversible exclusion or enforcement decisions.

## Relationship to HCS-26

- **HCS-26** defines the skill publication model and release artifacts.
- **HCS-28** is a standalone trust-scoring standard for those skill releases.

HCS-28 defines its own execution profiles, adapter contract, contribution semantics, and output model. Implementations MUST conform to this document directly rather than assuming compatibility with any other trust-score specification.

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

### Baseline Signal Semantics

The baseline adapters depend on common input semantics. Conforming implementations MUST apply the following definitions:

1. **`upvotes`**
   - MUST be a non-negative integer.
   - MUST represent the count of active, unique voter assertions for the exact skill subject `(network, name, version)`.
   - A voter identity MUST be deduplicated to at most one active upvote per subject.
   - Removed/retracted votes MUST NOT be counted.

2. **`verified`**
   - MUST be version-scoped and MUST NOT be inferred across different versions of the same skill name.
   - MUST represent explicit verification approval status for the exact subject.

3. **Verification signals**
   - `signals.publisherBound.ok`, `signals.repoCommitIntegrity.ok`, `signals.manifestIntegrity.ok`, and `signals.domainProof.ok` MUST be booleans.
   - Missing signals MUST be treated as `false` in the baseline profile unless a profile variant explicitly defines a different rule.

4. **Metadata fields**
   - `description`, `homepage`, `repo`, `commit`, `category`, and `tags` MUST be read from normalized metadata (trimmed strings; tags with empty entries removed).
   - Empty strings MUST be treated as missing.

5. **Repository health inputs**
   - Repository-derived inputs MUST be sourced from public repository metadata or equivalent verifiable records.
   - Implementations MUST apply anti-abuse controls before converting repository inputs into `repository.health.score`.

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

Each externally reported score key (other than `total`) MUST have exactly one owning adapter. Implementations MUST NOT collapse multiple independently reported scores into a single adapter definition.

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

1. Clamp each emitted component to `[0,100]`.
2. For configured denominator adapters, missing required components MUST be materialized as `0`.
3. Compute each adapter total as arithmetic mean of that adapter’s component values (single-component adapters resolve to that component value).
4. Compute composite total as weighted mean of adapter totals:

```
total = round2( sum( weight_i * adapterTotal_i ) / sum(weight_i) )
```

If the denominator is empty, total MUST be `0`.

### Baseline Adapter Catalog

This catalog is the interoperable minimum for HCS-28. Each listed score has its own adapter.
Conforming implementations MUST implement each baseline adapter below unless they declare a profile variance and a scoring configuration version that omits it.

Per-adapter normative definitions are in the HCS-28 adapter catalog:

- [`hcs-28/adapters/index.md`](./hcs-28/adapters/index.md)
- [`hcs-28/adapters.md`](./hcs-28/adapters.md)

#### Verification Adapters

All verification adapters apply in all profiles and emit a single `score` component.
Each verification adapter MUST emit exactly one key named `<adapterId>.score`.

| Adapter ID | Default Weight | Expected Input | Scoring Rule |
| --- | --- | --- | --- |
| `verification.review-status` | `0.50` | `verified` boolean | `100` when verified, else `0` |
| `verification.publisher-bound` | `0.20` | `signals.publisherBound.ok` | `100` when true, else `0` |
| `verification.repo-commit-integrity` | `0.40` | `signals.repoCommitIntegrity.ok` | `100` when true, else `0` |
| `verification.manifest-integrity` | `0.30` | `signals.manifestIntegrity.ok` | `100` when true, else `0` |
| `verification.domain-proof` | `0.10` | `signals.domainProof.ok` | `100` when true, else `0` |

If verification signals are missing, adapters MUST emit `0` unless an implementation explicitly enables a backward-compatibility mode and documents it via scoring configuration versioning.

#### Metadata Adapters

All metadata adapters apply in all profiles and emit a single `score` component.
Each metadata adapter MUST emit exactly one key named `<adapterId>.score`.

| Adapter ID | Default Weight | Scoring Rule |
| --- | --- | --- |
| `metadata.links` | `0.30` | `100` when both homepage and repo are present, `60` when either is present, else `0` |
| `metadata.description` | `0.25` | `>=160 => 100`, `>=80 => 85`, `>=30 => 65`, `>=10 => 40`, else `0` |
| `metadata.taxonomy` | `0.20` | Based on category presence and non-empty tag count |
| `metadata.provenance` | `0.25` | `100` for repo+commit, `70` repo only, `40` commit only, else `0` |

#### Upvotes Adapter

- `id`: `upvotes`
- `weight`: `1.00`
- `contributionMode`: `conditional`
- Components: `upvotes.score`
- Applies in all profiles.
- Score function:

```
score = clamp( round( 100 * (1 - e^(-upvotes / 20)) ), 0, 100 )
```

Where `upvotes` uses the normative input definition in [Baseline Signal Semantics](#baseline-signal-semantics).

#### Safety Adapter

- `id`: `safety.cisco-scan`
- `weight`: `1.00`
- `contributionMode`: `universal`
- Components: `safety.cisco-scan.score`
- Applies in all profiles.

Behavior:

- In read profile, implementations SHOULD use persisted scan outcomes.
- In refresh profile, implementations MAY execute a safety scan.
- Scan output MUST be normalized to `[0,100]` and SHOULD be persisted for read-time reuse.

#### Repository Health Adapter

- `id`: `repository.health`
- `weight`: `1.00`
- `contributionMode`: `conditional`
- Components: `repository.health.score`
- Applies only when external lookups are enabled and a valid public source repository URL exists.

This adapter MUST use a generalized anti-gaming model and MUST NOT expose raw, directly gameable sub-metrics as separately weighted trust adapters. Implementations SHOULD include anti-abuse penalties, temporal decay, and signal smoothing.

Optional derived convenience fields such as `verification.score` and `metadata.score` MAY be published, but they are aggregation outputs and MUST NOT replace the required per-score adapters above.

### Output Schema

A conforming response MUST include:

- `trustScores.total` in `[0,100]`, and
- baseline adapter outputs keyed as `<adapterId>.score` for all applicable adapters.
- optional additional per-component entries keyed as `<adapterId>.<componentKey>`.

Example:

```json
{
  "trustScores": {
    "upvotes.score": 28.19,
    "verification.review-status.score": 100,
    "verification.publisher-bound.score": 100,
    "verification.repo-commit-integrity.score": 100,
    "verification.manifest-integrity.score": 100,
    "verification.domain-proof.score": 0,
    "metadata.links.score": 100,
    "metadata.description.score": 85,
    "metadata.taxonomy.score": 70,
    "metadata.provenance.score": 100,
    "safety.cisco-scan.score": 94,
    "repository.health.score": 72.34,
    "verification.score": 90,
    "metadata.score": 88.25,
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

An implementation conforms to HCS-28 if it:

1. Implements the adapter contract and contribution semantics in this document.
2. Produces clamped `[0,100]` component values and weighted composite totals.
3. Supports the baseline adapter set defined above (or clearly declares omitted adapters and resulting profile variance).
4. Exposes an explainable component breakdown with the total score.

## References

- HCS-26: Decentralized Agent Skills Registry (`./hcs-26.md`)
- RFC 2119 (`https://www.rfc-editor.org/rfc/rfc2119`)

## License

Apache-2.0
