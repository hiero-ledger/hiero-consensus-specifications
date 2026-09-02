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

- non-empty `tags[]` count (numeric taxonomy ids)
- non-empty `languages[]` count

## Normalization

- `score = 100` when tag count `>= 3` and language count `>= 1`
- `score = 85` when tag count `>= 1` and language count `>= 1`
- `score = 70` when tag count `>= 3` and language count `= 0`
- `score = 55` when tag count `>= 1` and language count `= 0`
- `score = 35` when tag count `= 0` and language count `>= 1`
- `score = 0` when tag count `= 0` and language count `= 0`

Implementations MUST emit `metadata.taxonomy.score` as a finite value in `[0,100]`.

## Evidence (Recommended)

Implementations SHOULD preserve the counts used for scoring, for example:

```json
{
  "tagCount": 3,
  "languageCount": 1
}
```
