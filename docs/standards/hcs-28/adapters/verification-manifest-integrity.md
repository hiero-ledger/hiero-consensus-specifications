# HCS-28 (Adapter): `verification.manifest-integrity`

## Purpose

Reward successful manifest checksum verification for a published skill version.

## Contribution

- Adapter id: `verification.manifest-integrity`
- Contribution mode: `conditional`
- Default weight: `0.30`
- Output key: `verification.manifest-integrity.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `signals.manifestIntegrity.ok` boolean.

## Normalization

- `score = 100` when `signals.manifestIntegrity.ok=true`
- `score = 0` otherwise

Implementations MUST emit `verification.manifest-integrity.score` as a finite value in `[0,100]`.
