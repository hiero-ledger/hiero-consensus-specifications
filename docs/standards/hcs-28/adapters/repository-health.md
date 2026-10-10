# HCS-28 (Adapter): `repository.health`

## Purpose

Estimate repository trustworthiness while resisting direct metric gaming.

## Contribution

- Adapter id: `repository.health`
- Contribution mode: `conditional`
- Default weight: `1.00`
- Output key: `repository.health.score`

## Applicability

Applies when the skill declares a valid public source repository URL.

If `includeExternal=false`, implementations MAY return a persisted score if available. If no persisted score exists and external I/O is disabled, implementations SHOULD return no score (so a conditional adapter does not contribute).

## Inputs

- repository URL (`repo`) and optional pinned commit (`commit`);
- repository metadata from a public host (for example, GitHub);
- a profile-defined comparable repository cohort;
- freshness timestamp.

If the repository host is not supported or cannot be queried deterministically, the adapter SHOULD emit no score (so a conditional adapter does not contribute) and SHOULD provide evidence indicating `not_applicable`.

## Normalization

This adapter MUST expose only `repository.health.score` as its externally weighted score.

For baseline interoperability, implementations MUST compute `repository.health.score` deterministically from repository-derived signals while resisting direct metric gaming.

At minimum, the implementation MUST derive a score from a subset of the following inputs when available:

- `isArchived` (boolean)
- `stars` (non-negative integer)
- `watchers` or equivalent subscriber count (non-negative integer)
- `openIssues` (non-negative integer)
- `ageDays` (non-negative integer)
- `daysSinceLastPush` (non-negative integer)
- engagement or watcher-to-star ratio

If `isArchived=true`, `score` MUST be `0`.

Otherwise:

1. The implementation MUST derive monotonic sub-signals for repository popularity, community engagement, maintenance recency, and resilience / issue burden.
2. Each sub-signal MUST be normalized relative to a comparable cohort defined by the scoring profile.
3. Implementations MAY use log scaling before relative normalization.
4. When the cohort is large enough for stable distribution statistics, implementations MAY use bell-curve, z-score, or equivalent distribution-based normalization.
5. When the cohort is too small for stable distribution statistics, implementations MUST use a deterministic percentile or rank-based fallback instead of collapsing scores to a fixed neutral value.
6. If the cohort contains exactly one applicable repository and no disqualifying state such as `isArchived=true` applies, the implementation MAY assign `100` when the subject is the sole cohort leader.
7. The final score MUST be a weighted combination of the normalized sub-signals and MUST be clamped to `[0,100]`.

Implementations MUST NOT expose raw, directly gameable repository sub-metrics as separate weighted trust adapters in the HCS-28 baseline profile.

Implementations MUST emit `repository.health.score` as a finite value in `[0,100]`.

## Freshness (Baseline)

In baseline interoperability mode, persisted repository health results SHOULD expire after 7 days.

## Evidence (Recommended)

Implementations SHOULD preserve the derived inputs used for scoring, for example:

```json
{
  "isArchived": false,
  "stars": 50,
  "watchers": 12,
  "openIssues": 10,
  "ageDays": 420,
  "daysSinceLastPush": 30,
  "cohortSize": 37,
  "normalizationMode": "bell_curve",
  "computedAt": "2026-03-02T12:00:00.000Z"
}
```
