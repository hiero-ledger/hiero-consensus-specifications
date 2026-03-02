# HCS-28 (Adapter): `metadata.links`

## Purpose

Measure link completeness in skill metadata.

## Contribution

- Adapter id: `metadata.links`
- Contribution mode: `conditional`
- Default weight: `0.30`
- Output key: `metadata.links.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `homepage` presence
- `repo` presence

## Normalization

- `score = 100` when both `homepage` and `repo` are present
- `score = 60` when either one is present
- `score = 0` when neither is present

Implementations MUST emit `metadata.links.score` as a finite value in `[0,100]`.
