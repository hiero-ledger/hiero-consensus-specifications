# HCS-28 (Adapter): `upvotes`

## Purpose

Map community upvotes into a bounded, diminishing-return score.

## Contribution

- Adapter id: `upvotes`
- Contribution mode: `conditional`
- Default weight: `1.00`
- Output key: `upvotes.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- non-negative integer `upvotes`.

## Normalization

```
score = clamp( round(100 * (1 - e^(-upvotes / 20))), 0, 100 )
```
