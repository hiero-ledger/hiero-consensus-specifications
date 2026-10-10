# HCS-28 Profile: Runtime Security v1

## Status

Draft.

## Profile Identity

- Profile id: `hcs-28/runtime-security`
- Profile version: `1.0`
- Composite range: `[0,100]`
- Rounding precision: `0` decimal places

## Purpose

This profile prioritizes security, runtime exposure, artifact integrity, reviewability, and prompt-context cost for an exact skill release. It intentionally excludes popularity, repository activity, metadata description length, and taxonomy completeness from the composite score because those signals do not directly establish runtime safety.

The profile is an optional HCS-28 scoring profile. Supporting it does not replace the `hcs-28/baseline` interoperability requirement.

## Fixed Factor Weights

All factors use `scoped` contribution mode. The denominator is always `1.00`; a missing factor MUST contribute `0` and MUST NOT be averaged away.

| Factor adapter id | Weight | Output key |
| --- | ---: | --- |
| `evidence.security` | `0.35` | `evidence.security.score` |
| `evidence.permissions` | `0.20` | `evidence.permissions.score` |
| `evidence.integrity` | `0.20` | `evidence.integrity.score` |
| `evidence.provenance` | `0.10` | `evidence.provenance.score` |
| `evidence.usability` | `0.10` | `evidence.usability.score` |
| `evidence.context-cost` | `0.05` | `evidence.context-cost.score` |

The composite MUST be computed as:

```text
total = round0(
  0.35 * evidence.security.score +
  0.20 * evidence.permissions.score +
  0.20 * evidence.integrity.score +
  0.10 * evidence.provenance.score +
  0.10 * evidence.usability.score +
  0.05 * evidence.context-cost.score
)
```

Implementations MUST also report `evidenceCoveragePercent` as the sum of weights for present factors multiplied by `100`. Coverage MUST NOT alter the composite denominator.

## Evidence Binding

Every factor MUST be scoped to the exact skill subject and version. Content-derived evidence MUST include a cryptographic content hash. Server-derived content measurements MUST match the stored bytes for that hash.

Client-submitted scanner results MUST NOT be treated as authoritative. An implementation MAY consume a legacy server-run scan that lacks exact-byte input evidence only when the scan record is bound to the same subject content hash. It MUST label the evidence `legacy_unbound`, assign confidence no greater than `60`, and MUST NOT reuse it across content versions.

When multiple authoritative scanner results exist for the same factor and content hash, only the newest result MUST contribute components and score. Profile version `1.0` does not expire a result solely because of age. Implementations MAY apply a stricter freshness policy only through a separately versioned profile variance.

## Safety Scan

`evidence.security` is profile-scoped and distinct from the baseline `safety.cisco-scan` adapter. It evaluates the server-run scan of the stored primary skill instruction file for the exact content hash; it does not imply that every supplementary file in an HCS-26 manifest was scanned.

```text
score = clamp(
  100 - (35 * critical + 25 * high + 12 * medium + 5 * low),
  0,
  100
)
```

The scan MUST cover the exact stored primary instruction bytes. No authoritative scan yields `0` and zero coverage for this factor.

## Permission and Egress Exposure

`evidence.permissions` evaluates declared behavior, not observed runtime activity. Inputs MUST come from signed inventory or high-confidence skill-documentation analysis. Unknown, unsigned, or empty capability evidence yields `0` and zero coverage; it MUST NOT be interpreted as absence of risk.

For known capability evidence:

```text
score = clamp(100 - sum(deduction(capability)), 0, 100)
```

| Capability | Deduction |
| --- | ---: |
| `loads_remote_code` | `50` |
| `executes_code` | `50` |
| `reads_secrets` | `45` |
| `changes_permissions` | `45` |
| `deletes_files` | `40` |
| `runs_shell` | `35` |
| `writes_files` | `25` |
| `network_egress` | `20` |
| `network_ingress` | `20` |
| `uses_clipboard` | `20` |
| `uses_browser` | `15` |
| `posts_messages` | `10` |
| `uses_model_sampling` | `10` |
| `reads_files` | `5` |
| `reads_messages` | `5` |

Implementations MUST version additions or deduction changes. Unknown or unrecognized values MUST NOT receive a default deduction. When at least one recognized capability exists, unknown values are ignored and SHOULD be listed in evidence metadata. When no recognized capability exists, the factor status is `missing`, its score is `0`, and it contributes zero coverage.

## Artifact Integrity

`evidence.integrity` combines three HCS-28 verification adapters with a fixed denominator. Missing sub-signals MUST materialize as `0`:

```text
score = round0(
  (0.50 * verification.review-status.score +
   0.40 * verification.repo-commit-integrity.score +
   0.30 * verification.manifest-integrity.score) / 1.20
)
```

The factor has coverage when at least one sub-signal is present. Missing sub-signals still contribute `0` to the fixed sub-signal denominator.

## Publisher and Source Provenance

`evidence.provenance` combines publisher, domain, and metadata provenance with a fixed denominator:

```text
score = round0(
  (0.20 * verification.publisher-bound.score +
   0.10 * verification.domain-proof.score +
   0.25 * metadata.provenance.score) / 0.55
)
```

The factor has coverage when at least one sub-signal is present. Missing sub-signals MUST contribute `0`.

## Instruction Quality

`evidence.usability` applies only when the implementation has server-verified stored bytes and content-bound `lineCount`, `headingCount`, `frontmatterPresent`, and `readabilityStatus` evidence. Missing required metrics yields `0` and zero coverage.

Unreadable or empty content scores `0`. When the bytes and all required metrics are present, an unreadable result still counts as present evidence and contributes this factor's weight to coverage. Otherwise:

```text
score = 40
score += 20 when frontmatter is present
score += min(headingCount * 5, 20)
score += 20 when 10 <= lineCount <= 400
score += 15 when the previous condition is false and lineCount <= 1000
score += 5 otherwise
score = clamp(score, 0, 100)
```

## Context Cost

`evidence.context-cost` applies only to server-verified stored bytes. Missing exact-byte evidence yields `0` and zero coverage.

Implementations MUST estimate prompt tokens as:

```text
estimatedTokens = ceil(utf8ByteLength / 4)
```

| Estimated tokens | Score |
| --- | ---: |
| `< 128` | `35` |
| `128-255` | `60` |
| `256-2,000` | `100` |
| `2,001-4,000` | `85` |
| `4,001-8,000` | `60` |
| `8,001-16,000` | `35` |
| `> 16,000` | `15` |

This factor measures likely prompt-context cost and reviewability. It is not a tokenizer-exact billing estimate.

## Required Output Metadata

In addition to the HCS-28 output schema, conforming profile output MUST include:

- `profile.id = "hcs-28/runtime-security"`;
- `profile.version = "1.0"`;
- `evidenceCoveragePercent` in `[0,100]`;
- per-factor score, weight, status, confidence, and evidence binding; and
- the content hash used for content-bound factors.

Factor status MUST distinguish `missing`, `verified`, `warning`, and `critical`. Confidence MUST describe evidence authority and MUST NOT be multiplied into the factor score unless a future profile version explicitly changes the aggregation rule. Internal component objects or profile aliases do not constitute the HCS-28 wire representation; conforming exporters MUST map them to the profile and score keys above.

## Test Vectors

### Complete Evidence

Given:

- scanner counts `critical = 0`, `high = 0`, `medium = 0`, and `low = 1`, producing `evidence.security.score = 95`;
- declared capabilities `network_egress` and `reads_files`, producing `evidence.permissions.score = 75`;
- all three artifact-integrity sub-signals equal `100`, producing `evidence.integrity.score = 100`;
- publisher-bound `100`, domain-proof `0`, and metadata-provenance `100`, producing `evidence.provenance.score = 82`;
- `7,131` server-verified bytes, `198` lines, `15` headings, readable status, and frontmatter, producing `evidence.usability.score = 100`; and
- `estimatedTokens = ceil(7,131 / 4) = 1,783`, producing `evidence.context-cost.score = 100`.

Then:

```text
total = round0(
  95 * 0.35 +
  75 * 0.20 +
  100 * 0.20 +
  82 * 0.10 +
  100 * 0.10 +
  100 * 0.05
) = 91
```

All six factors are present, so `evidenceCoveragePercent = 100`.

### Missing Evidence

Given no authoritative scanner result, unknown capability evidence, no verification or provenance signals, and device-claimed bytes that have not been verified against stored content, all six factors MUST score `0`, `total` MUST equal `0`, and `evidenceCoveragePercent` MUST equal `0`.

## Security Considerations

- Implementations MUST recompute current views when newer authoritative evidence or capability declarations exist for the same content hash.
- Stored aggregate scores MUST NOT take precedence over newer factor evidence.
- Scanner layers with mismatched item identity or content hash MUST be rejected.
- Device-claimed byte length MUST NOT contribute to instruction-quality or context-cost scores until verified against stored bytes.
- Evidence absence MUST NOT be presented as evidence of safety.
