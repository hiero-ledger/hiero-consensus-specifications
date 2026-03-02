# HCS-28 (Adapter): `metadata.description`

## Purpose

Measure descriptive quality using deterministic length thresholds.

## Contribution

- Adapter id: `metadata.description`
- Contribution mode: `conditional`
- Default weight: `0.25`
- Output key: `metadata.description.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `description` length in characters after trim.

## Normalization

- `score = 100` when length `>= 160`
- `score = 85` when length `>= 80`
- `score = 65` when length `>= 30`
- `score = 40` when length `>= 10`
- `score = 0` otherwise
