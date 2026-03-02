# HCS-28 (Adapter): `verification.domain-proof`

## Purpose

Reward successful domain-control proof for the skill publisher.

## Contribution

- Adapter id: `verification.domain-proof`
- Contribution mode: `conditional`
- Default weight: `0.10`
- Output key: `verification.domain-proof.score`

## Applicability

Applies when a skill declares a domain/homepage or when a domain proof signal exists.

## Inputs

- `signals.domainProof.ok` boolean.

## Normalization

- `score = 100` when `signals.domainProof.ok=true`
- `score = 0` otherwise
