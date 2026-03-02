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

- non-negative integer `upvotes`, defined as active unique upvotes for the exact skill subject `(network, discovery_topic_id, skill_uid, version)`.
- each voter identity MUST contribute at most one active upvote to the count.
- retracted votes MUST be excluded.

## Normalization

```
score = clamp( round(100 * (1 - e^(-upvotes / 20))), 0, 100 )
```

Implementations MUST emit a finite numeric value for `upvotes.score` and MUST clamp to `[0,100]`.

`round(x)` rounds to the nearest integer using half-up rounding.
