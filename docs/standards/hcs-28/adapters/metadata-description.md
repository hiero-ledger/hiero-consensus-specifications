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

Character counting MUST be deterministic. In baseline interoperability mode, implementations SHOULD count Unicode code points after trimming leading and trailing whitespace.

## Normalization

- `score = 100` when length `>= 160`
- `score = 85` when length `>= 80`
- `score = 65` when length `>= 30`
- `score = 40` when length `>= 10`
- `score = 0` otherwise

Implementations MUST emit `metadata.description.score` as a finite value in `[0,100]`.

## Evidence (Recommended)

Implementations SHOULD preserve the normalized length used for scoring, for example:

```json
{
  "descriptionLength": 182
}
```
