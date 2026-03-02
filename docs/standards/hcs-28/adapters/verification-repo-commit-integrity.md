# HCS-28 (Adapter): `verification.repo-commit-integrity`

## Purpose

Reward cryptographic integrity between declared repository commit content and published skill artifact hashes.

## Contribution

- Adapter id: `verification.repo-commit-integrity`
- Contribution mode: `conditional`
- Default weight: `0.40`
- Output key: `verification.repo-commit-integrity.score`

## Applicability

Applies when repository and commit metadata are present or when an integrity signal exists.

## Inputs

- `signals.repoCommitIntegrity.ok` boolean.

## Normalization

- `score = 100` when `signals.repoCommitIntegrity.ok=true`
- `score = 0` otherwise

Implementations MUST emit `verification.repo-commit-integrity.score` as a finite value in `[0,100]`.
