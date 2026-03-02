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

Implementations MUST emit `verification.domain-proof.score` as a finite value in `[0,100]`.

## Semantics

`signals.domainProof.ok` MUST be `true` only when a verifier has established control of a domain associated with the skill (typically derived from `homepage`) using a verifiable challenge-response method such as:

- DNS TXT record challenge, or
- HTTPS well-known file challenge.

If no domain is declared, if the proof cannot be verified, or if the proof is expired or malformed, `signals.domainProof.ok` MUST be `false`.
