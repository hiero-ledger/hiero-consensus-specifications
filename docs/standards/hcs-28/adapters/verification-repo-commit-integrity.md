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

## Semantics

`signals.repoCommitIntegrity.ok` MUST be `true` only when all of the following hold:

1. The skill declares `repo` and `commit` metadata (via HCS-26 discovery metadata and/or the `SKILL.json` manifest).
2. The verifier can fetch the repository contents at the declared commit.
3. For each file entry in `SKILL.json.files[]`, the verifier can locate the corresponding file content at the declared commit and the computed `sha256` matches the manifest `files[].sha256`.

If `repo` or `commit` is missing, if the repository is private/unreachable, or if any file is missing or mismatched, `signals.repoCommitIntegrity.ok` MUST be `false`.
