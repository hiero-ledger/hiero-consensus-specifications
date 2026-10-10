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
- voter identities MUST be represented by a stable `voter_id` string (scheme defined by the implementation and versioned as part of the scoring configuration).
- each counted upvote MUST be attributable to a voter identity. Anonymous or unverifiable votes MUST NOT be counted in baseline interoperability mode.
- retracted votes MUST be excluded.

An implementation MAY store votes in any form, but the resulting `upvotes` count MUST be equivalent to:

> count(distinct voter_id) where latest_vote_state(voter_id, subject) = active

## Evidence (Recommended)

To support explainability, implementations SHOULD preserve the `upvotes` count used for scoring and a timestamp of when it was computed, for example:

```json
{
  "upvotes": 23,
  "computedAt": "2026-03-02T12:00:00.000Z"
}
```

## Normalization

```
score = clamp( round(100 * (1 - e^(-upvotes / 20))), 0, 100 )
```

Implementations MUST emit a finite numeric value for `upvotes.score` and MUST clamp to `[0,100]`.

`round(x)` rounds to the nearest integer using half-up rounding.
