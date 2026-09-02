# HCS-28 (Adapter): `verification.review-status`

## Purpose

Represent version-scoped verification review state as a trust signal.

## Contribution

- Adapter id: `verification.review-status`
- Contribution mode: `conditional`
- Default weight: `0.50`
- Output key: `verification.review-status.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `verified` boolean for the exact `(network, discovery_topic_id, skill_uid, version)` subject.

The `verified` value MUST represent an explicit approval state for that subject and MUST NOT be inferred or copied across versions.

## Normalization

- `score = 100` when `verified=true`
- `score = 0` when `verified=false` or missing

Implementations MUST emit `verification.review-status.score` as a finite value in `[0,100]`.
