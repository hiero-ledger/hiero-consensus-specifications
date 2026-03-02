# HCS-28 (Adapter): `metadata.provenance`

## Purpose

Measure source provenance quality from repository and commit declarations.

## Contribution

- Adapter id: `metadata.provenance`
- Contribution mode: `conditional`
- Default weight: `0.25`
- Output key: `metadata.provenance.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `repo` presence
- `commit` presence

## Normalization

- `score = 100` when both `repo` and `commit` exist
- `score = 70` when `repo` exists and `commit` missing
- `score = 40` when `commit` exists and `repo` missing
- `score = 0` otherwise
