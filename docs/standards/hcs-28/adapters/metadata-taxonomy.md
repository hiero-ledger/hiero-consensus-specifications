# HCS-28 (Adapter): `metadata.taxonomy`

## Purpose

Measure how well a skill is categorized and tagged for discovery.

## Contribution

- Adapter id: `metadata.taxonomy`
- Contribution mode: `conditional`
- Default weight: `0.20`
- Output key: `metadata.taxonomy.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `category` presence
- non-empty `tags[]` count

## Normalization

- `score = 100` when category exists and tag count `>= 3`
- `score = 85` when category exists and tag count `>= 1`
- `score = 60` when category exists and tag count `= 0`
- `score = 55` when category missing and tag count `>= 3`
- `score = 35` when category missing and tag count `>= 1`
- `score = 0` otherwise
