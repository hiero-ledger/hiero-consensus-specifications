---
title: HCS-28 - Skill Trust Scores and Adapters
description: A deterministic trust-scoring standard for HCS-26 skill releases, including independent scoring profiles, adapter interfaces, and aggregation rules.
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
  - [Scope](#scope)
  - [Subject Model](#subject-model)
  - [Baseline Signal Semantics](#baseline-signal-semantics)
  - [Scoring Profiles](#scoring-profiles)
  - [Adapter Contract](#adapter-contract)
  - [Adapter Identifier Namespacing](#adapter-identifier-namespacing)
  - [Contribution Modes](#contribution-modes)
  - [Execution Modes](#execution-modes)
  - [Freshness and Staleness](#freshness-and-staleness)
  - [Normalization and Aggregation](#normalization-and-aggregation)
  - [Baseline Adapter Catalog](#baseline-adapter-catalog)
    - [Verification Adapters](#verification-adapters)
    - [Metadata Adapters](#metadata-adapters)
    - [Upvotes Adapter](#upvotes-adapter)
    - [Safety Adapter](#safety-adapter)
    - [Repository Health Adapter](#repository-health-adapter)
  - [Output Schema](#output-schema)
  - [Reputation Topic Publication (Optional)](#reputation-topic-publication-optional)
  - [Test Vectors](#test-vectors)
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

- **Context-bound** (subject + execution mode + adapter configuration version),
- **Derived and versioned** (signal freshness and adapter logic matter), and
- **Decision support** (one input among policy, provenance, and human review).

Implementations MUST NOT use HCS-28 totals as the sole authoritative basis for irreversible exclusion or enforcement decisions.

## Relationship to HCS-26

- **HCS-26** defines the skill publication model and release artifacts.
- **HCS-28** is a standalone trust-scoring standard for those skill releases.

HCS-28 defines its own execution modes, adapter contract, contribution semantics, and output model. Implementations MUST conform to this document directly rather than assuming compatibility with any other trust-score specification.

## Terminology

- **Skill Subject**: An HCS-26 skill release identified by `(network, discovery_topic_id, skill_uid, version)`.
- **Discovery Topic ID**: HCS-2 topic id of the HCS-26 discovery registry.
- **Skill UID**: Sequence number of the HCS-26 discovery registry `register` message (see HCS-26).
- **Version**: Semantic version string for a skill version entry (see HCS-26).
- **Manifest**: The `SKILL.json` manifest referenced by the skill version (see HCS-26).
- **Trust Adapter**: Deterministic function mapping skill data/signals to normalized components.
- **Component**: A normalized value in `[0,100]` emitted by one adapter.
- **Adapter Total**: Mean of components emitted by one adapter.
- **Composite Skill Trust Score**: Weighted mean of adapter totals in `[0,100]`.
- **Scoring Profile**: A named scoring configuration defining adapter membership, weights, and normalization rules.
- **Execution Mode**: Runtime mode deciding whether external I/O is allowed (for example, read vs refresh).

Normative language such as **MUST**, **SHOULD**, and **MAY** follows RFC 2119.

## Specification

### Scope

HCS-28 defines:

- a deterministic adapter contract for skill trust scoring;
- how to normalize adapter outputs into `[0,100]`;
- how to aggregate adapter totals into a composite score; and
- a baseline adapter catalog and scoring profile for interoperability.

HCS-28 does not mandate a specific registry implementation, storage layer, refresh schedule, voting mechanism, or verification governance model.

### Subject Model

A conforming implementation MUST compute trust for one skill release record containing:

- `network`, `discovery_topic_id`, `skill_uid`, `version`,
- release metadata (HCS-26 discovery metadata and/or the HCS-26 `SKILL.json` manifest fields),
- published files from `SKILL.json` (including file paths and either `sha256` values or deterministic artifact references sufficient to recover the published bytes),
- verification fields (`verified`, optional verification signals),
- safety fields (optional persisted safety summary / findings),
- voting context (`upvotes`).

Implementations MUST treat a skill subject as the tuple `(network, discovery_topic_id, skill_uid, version)` for uniqueness.

### Baseline Signal Semantics

The baseline adapters depend on common input semantics. Conforming implementations MUST apply the following definitions:

1. **`upvotes`**
   - MUST be a non-negative integer.
   - MUST represent the count of active, unique voter assertions for the exact skill subject `(network, discovery_topic_id, skill_uid, version)`.
   - A voter identity MUST be represented by a stable `voter_id` string and MUST be deduplicated to at most one active upvote per subject.
   - Implementations MUST define the `voter_id` scheme used (for example, `hedera:0.0.1234` or `evm:0xabc...`) and MUST version it as part of the scoring configuration.
   - An upvote assertion MUST be attributable to a voter identity. Votes from unverifiable or anonymous identities MUST NOT be counted in baseline interoperability mode.
   - Removed/retracted votes MUST NOT be counted.

2. **`verified`**
   - MUST be version-scoped and MUST NOT be inferred across different versions of the same skill name.
   - MUST represent explicit verification approval status for the exact subject.

3. **Verification signals**
   - `signals.publisherBound.ok`, `signals.repoCommitIntegrity.ok`, `signals.manifestIntegrity.ok`, and `signals.domainProof.ok` MUST be booleans.
   - Missing signals MUST be treated as `false` in the baseline profile unless a profile variant explicitly defines a different rule.

   For interoperability, a verification signal object SHOULD use the following shape:

   ```json
   {
     "signals": {
       "publisherBound": { "ok": true, "checkedAt": "2026-03-02T12:00:00.000Z" },
       "repoCommitIntegrity": { "ok": true, "checkedAt": "2026-03-02T12:00:00.000Z" },
       "manifestIntegrity": { "ok": true, "checkedAt": "2026-03-02T12:00:00.000Z" },
       "domainProof": { "ok": false, "checkedAt": "2026-03-02T12:00:00.000Z", "reason": "no_homepage" }
     }
   }
   ```

   If a signal can expire (for example, domain control proof), the signal SHOULD include an `expiresAt` ISO-8601 timestamp.
   If `expiresAt` is present and is in the past at computation time, implementations MUST treat the signal as `ok=false`.

4. **Metadata fields**
   - `description`, `homepage`, `repo`, and `commit` MUST be read from normalized metadata (trimmed strings).
   - Empty strings MUST be treated as missing.
   - `tags` MUST be treated as a set of numeric taxonomy ids (HCS-26 uses OASF skill ids).
   - `languages` MUST be treated as a set of normalized language identifiers:
     - values MUST be trimmed and lowercased,
     - duplicates MUST be removed,
     - implementations SHOULD use BCP-47 or ISO-639 language identifiers where available.

5. **Repository health inputs**
   - Repository-derived inputs MUST be sourced from public repository metadata or equivalent verifiable records.
   - Implementations MUST apply anti-abuse controls before converting repository inputs into `repository.health.score`.
   - Implementations MUST normalize repository health relative to a comparable cohort defined by the scoring profile. They MUST NOT rely exclusively on fixed absolute star-count thresholds in baseline interoperability mode.
   - If the applicable cohort is too small for a stable distribution fit, implementations MUST use a deterministic fallback that preserves ordering across the available cohort. They MUST NOT collapse all small-cohort subjects to a fixed neutral score solely because the sample size is small.

### Scoring Profiles

Conforming implementations MUST expose scores for at least one profile:

- `hcs-28/baseline`

A scoring profile MUST define:

- adapter membership;
- weights and contribution modes; and
- interpretation rules for missing and stale data.

Implementations MAY define additional profiles, but MUST NOT claim baseline interoperability unless the `hcs-28/baseline` profile is supported.

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

#### Adapter output keys

Adapters MUST emit keys in the form:

```
<adapterId>.<componentKey>
```

For the baseline profile, `componentKey` is `score` and each adapter emits exactly one key named `<adapterId>.score`.

If `fetchScores(...)` returns `null`, the adapter emits no keys.

#### Context

The `context` object MUST include:

- `profileId` (string),
- `profileVersion` (string), and
- `includeExternal` (boolean).

### Adapter Identifier Namespacing

Adapter identifiers MUST be stable and lowercase. Identifiers SHOULD use dot-separated namespaces to avoid collisions (for example, `metadata.links`).

The baseline profile reserves the following identifier prefixes:

- `verification.*`
- `metadata.*`
- `upvotes`
- `safety.*`
- `repository.*`

Non-baseline adapters SHOULD be namespaced to an organization or implementation to reduce collision risk.

### Contribution Modes

The adapter contribution mode controls denominator behavior:

- `conditional` (default): adapter participates in denominator only if it emitted scores.
- `universal`: if applicable, adapter participates in denominator even when it emits no scores.
- `scoped`: same denominator behavior as universal; intended for profile-scoped policies.

### Execution Modes

A conforming implementation SHOULD support at least:

1. **Read mode** (`includeExternal=false`): external lookups disabled for low-latency responses.
2. **Refresh mode** (`includeExternal=true`): external lookups allowed for background trust refresh.

The same scoring/aggregation algorithm MUST apply to both modes; only adapter applicability/input freshness may differ.

When `includeExternal=false`, implementations MUST NOT perform network I/O as part of trust computation. Adapters MAY return persisted/cached results.

When `includeExternal=true`, implementations MAY perform network I/O, but MUST use bounded timeouts and MUST treat transient upstream failures as missing data rather than blocking score computation.

### Freshness and Staleness

Some adapters depend on time-varying external sources (for example, repository metadata and safety scanner outputs).
To keep scores reproducible, implementations MUST define freshness rules per scoring configuration version.

In baseline interoperability mode:

1. Adapters SHOULD persist time-varying results along with `computedAt` (ISO-8601).
2. Each persisted result SHOULD define an `expiresAt` timestamp, or the implementation MUST apply a profile-configured `maxAge`.
3. If `includeExternal=false` and the most recent persisted result is expired or older than `maxAge`, the adapter MUST behave as if no result exists:
   - conditional adapters emit no keys;
   - universal adapters emit required keys as `0`.
4. If `includeExternal=true`, adapters SHOULD refresh expired results. If refresh fails, adapters MAY fall back to a non-expired persisted result; otherwise, they MUST behave as if no result exists.

Baseline recommended `maxAge` values (implementations MAY override but MUST version overrides):

| Adapter ID | Recommended `maxAge` |
| --- | --- |
| `safety.cisco-scan` | 30 days |
| `repository.health` | 7 days |

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

#### Definitions

- `clamp(x, 0, 100) = min(100, max(0, x))`
- `round2(x)` rounds to 2 decimal places using half-up rounding.

#### Denominator selection

For a given scoring profile, the eligible adapter set is:

1. Adapters where `appliesTo(subject, context)` is true.
2. From that set:
   - `universal` and `scoped` adapters participate in the denominator even when they emit no keys (missing keys are treated as `0`).
   - `conditional` adapters participate in the denominator only when they emit at least one key.

### Baseline Adapter Catalog

This catalog is the interoperable minimum for HCS-28. Each listed score has its own adapter.
Conforming implementations MUST implement each baseline adapter below unless they declare a profile variance and a scoring configuration version that omits it.

Per-adapter normative definitions are in the HCS-28 adapter catalog:

- [`hcs-28/adapters/index.md`](./hcs-28/adapters/index.md)
- [`hcs-28/adapters.md`](./hcs-28/adapters.md)

#### Verification Adapters

All verification adapters apply in all execution modes and emit a single `score` component.
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

All metadata adapters apply in all execution modes and emit a single `score` component.
Each metadata adapter MUST emit exactly one key named `<adapterId>.score`.

| Adapter ID | Default Weight | Scoring Rule |
| --- | --- | --- |
| `metadata.links` | `0.30` | `100` when both homepage and repo are present, `60` when either is present, else `0` |
| `metadata.description` | `0.25` | `>=160 => 100`, `>=80 => 85`, `>=30 => 65`, `>=10 => 40`, else `0` |
| `metadata.taxonomy` | `0.20` | Based on tag count and language count (see adapter doc) |
| `metadata.provenance` | `0.25` | `100` for repo+commit, `70` repo only, `40` commit only, else `0` |

#### Upvotes Adapter

- `id`: `upvotes`
- `weight`: `1.00`
- `contributionMode`: `conditional`
- Components: `upvotes.score`
- Applies in all execution modes.
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
- Applies in all execution modes.

Behavior:

- In read mode, implementations SHOULD use persisted scan outcomes.
- In refresh mode, implementations MAY execute a safety scan.
- Scan output MUST be normalized to `[0,100]` and SHOULD be persisted for read-time reuse.

#### Repository Health Adapter

- `id`: `repository.health`
- `weight`: `1.00`
- `contributionMode`: `conditional`
- Components: `repository.health.score`
- Applies when a valid public source repository URL exists (and may rely on external lookup or persisted results).

This adapter MUST emit only `repository.health.score` as its baseline score key. Its deterministic baseline normalization is defined in the per-adapter catalog.

For baseline interoperability, the adapter score MUST remain cohort-relative even when the cohort is small. Implementations MAY switch from bell-curve normalization to percentile or rank-based normalization when the cohort is undersized, but they MUST keep the result deterministic and monotonic.

Optional derived convenience fields such as `verification.score` and `metadata.score` MAY be published, but they are aggregation outputs and MUST NOT replace the required per-score adapters above.

If published, these subtotals MUST be computed as weighted means over the corresponding baseline adapter scores:

- `verification.score` MUST be the weighted mean of:
  - `verification.review-status.score`
  - `verification.publisher-bound.score`
  - `verification.repo-commit-integrity.score`
  - `verification.manifest-integrity.score`
  - `verification.domain-proof.score`
- `metadata.score` MUST be the weighted mean of:
  - `metadata.links.score`
  - `metadata.description.score`
  - `metadata.taxonomy.score`
  - `metadata.provenance.score`

### Output Schema

A conforming response MUST include:

- `subject` identifying the scored skill subject,
- `profile` identifying the scoring profile,
- `execution` describing execution mode, and
- `trustScores` containing component scores and `total`.

The `trustScores` object MUST include:

- `trustScores.total` in `[0,100]`, and
- baseline adapter outputs keyed as `<adapterId>.score` for all applicable adapters.

Additional per-component entries MAY be included and MUST be keyed as `<adapterId>.<componentKey>`.

#### Subject

The `subject` object MUST include:

- `network` (string),
- `discovery_topic_id` (string, HCS-2 topic id),
- `skill_uid` (string, discovery register sequence number), and
- `version` (string, semver).

#### Profile

The `profile` object MUST include:

- `id` (string), and
- `version` (string).

#### Execution

The `execution` object MUST include:

- `includeExternal` (boolean), and
- `computedAt` (string, ISO-8601).

Example:

```json
{
  "subject": {
    "network": "testnet",
    "discovery_topic_id": "0.0.123456",
    "skill_uid": "42",
    "version": "1.0.0"
  },
  "profile": {
    "id": "hcs-28/baseline",
    "version": "0.1"
  },
  "execution": {
    "includeExternal": false,
    "computedAt": "2026-03-02T12:00:00.000Z"
  },
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
    "total": 77.23
  }
}
```

### Reputation Topic Publication (Optional)

HCS-26 defines an optional reputation topic type (`type = 2`). HCS-28 standardizes how skill trust scores MAY be published to such a topic for portability.

If an implementation publishes HCS-28 scores to an HCS-26 reputation topic, it MUST publish messages with:

- `p = "hcs-28"`,
- `op = "score"`,
- a subject reference, and
- a score payload.

#### Message schema

```json
{
  "p": "hcs-28",
  "op": "score",
  "subject": {
    "network": "mainnet",
    "discovery_topic_id": "0.0.123456",
    "skill_uid": "42",
    "version": "1.0.0"
  },
  "profile": {
    "id": "hcs-28/baseline",
    "version": "0.1"
  },
  "execution": {
    "includeExternal": true,
    "computedAt": "2026-03-02T12:00:00.000Z"
  },
  "scores": {
    "total": 87.12,
    "upvotes.score": 28.19,
    "verification.review-status.score": 100
  },
  "snapshot": "hcs://1/0.0.99999"
}
```

Rules:

1. The `subject` tuple MUST identify the exact skill version being scored.
2. The `scores.total` MUST be present.
3. `scores` MAY include additional adapter component keys. If it does not include the full set, the message SHOULD include `snapshot` referencing a complete score document stored in a decentralized file store (for example, HCS-1).
4. Consumers MUST treat on-topic `scores` as attestations. Authority, signature requirements, and consensus across multiple indexers are out of scope for HCS-28 and MAY follow HCS-26 reputation architectures.

### Test Vectors

These vectors are provided to make the baseline profile implementable and to reduce ambiguity in normalization and aggregation.

#### Upvotes normalization

Using `score = round(100 * (1 - e^(-upvotes / 20)))` with half-up rounding:

| `upvotes` | Expected `upvotes.score` |
| --- | --- |
| `0` | `0` |
| `1` | `5` |
| `5` | `22` |
| `10` | `39` |
| `20` | `63` |
| `50` | `92` |
| `100` | `99` |

#### Safety normalization

Given `raw = 100 - (30*critical + 12*high + 4*medium + 1*low)`:

| `critical` | `high` | `medium` | `low` | Expected `safety.cisco-scan.score` |
| --- | --- | --- | --- | --- |
| `0` | `0` | `0` | `0` | `100` |
| `1` | `0` | `0` | `0` | `70` |
| `0` | `1` | `3` | `10` | `66` |

#### Repository health normalization

Case A:

- `isArchived=false`, `isFork=false`
- `stars=50` → `starsScore=60`
- `daysSinceLastPush=30` → `recencyScore=85`
- `openIssues=10` → `issuesRatio=0.2` → `issuesScore=80`

Expected:

- `raw = 60*0.45 + 85*0.35 + 80*0.20 = 72.75`
- `repository.health.score = 72.75`

Case B:

- `isArchived=false`, `isFork=false`
- `stars=0` → `starsScore=0`
- `daysSinceLastPush=400` → `recencyScore=15`
- `openIssues=100` → `issuesRatio=100` → `issuesScore=20`

Expected:

- `raw = 0*0.45 + 15*0.35 + 20*0.20 = 9.25`
- `repository.health.score = 9.25`

#### Metadata description normalization

| `descriptionLength` | Expected `metadata.description.score` |
| --- | --- |
| `0` | `0` |
| `10` | `40` |
| `30` | `65` |
| `80` | `85` |
| `160` | `100` |

#### Verification signal mapping

| Input | Expected score |
| --- | --- |
| `verified=true` | `verification.review-status.score = 100` |
| `verified=false` | `verification.review-status.score = 0` |
| `signals.manifestIntegrity.ok=true` | `verification.manifest-integrity.score = 100` |
| `signals.manifestIntegrity.ok=false` or missing | `verification.manifest-integrity.score = 0` |

#### Aggregation (baseline weights)

If the following adapter scores are present:

- `upvotes.score = 28.19` (weight `1.00`)
- `verification.review-status.score = 100` (weight `0.50`)
- `metadata.links.score = 100` (weight `0.30`)
- `safety.cisco-scan.score = 94` (weight `1.00`)

Then:

- `denominator = 1.00 + 0.50 + 0.30 + 1.00 = 2.80`
- `numerator = 28.19*1.00 + 100*0.50 + 100*0.30 + 94*1.00 = 202.19`
- `total = round2(202.19 / 2.80) = 72.21`

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
