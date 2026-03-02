# HCS-28 (Adapter): `verification.publisher-bound`

## Purpose

Reward verified publisher identity binding for the skill version.

## Contribution

- Adapter id: `verification.publisher-bound`
- Contribution mode: `conditional`
- Default weight: `0.20`
- Output key: `verification.publisher-bound.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `signals.publisherBound.ok` boolean.

## Normalization

- `score = 100` when `signals.publisherBound.ok=true`
- `score = 0` otherwise

Implementations MUST emit `verification.publisher-bound.score` as a finite value in `[0,100]`.
