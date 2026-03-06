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
3. For each logical published file, the verifier can locate the corresponding repository content at the declared commit and reproduce the published artifact bytes deterministically.
4. The computed `sha256` of those published artifact bytes matches the skill’s published integrity source.

If `repo` or `commit` is missing, if the repository is private or unreachable, or if any required file is missing or mismatched, `signals.repoCommitIntegrity.ok` MUST be `false`.

File mapping rules (baseline):

- The verifier MUST treat `SKILL.json.files[].path` as a relative path from the repository root at the declared commit.
- The verifier MUST compute `sha256` over the published artifact bytes for that file.
- If the publication pipeline applies a deterministic transform before publication, the verifier MUST compare against the transformed bytes rather than the unmodified repository bytes.
- Deterministic publication transforms MUST be fully specified and reproducible. Examples include canonical JSON serialization, normalized path ordering, or injection of version / provenance fields from the release subject.
- The verifier MAY derive the expected published digest from either:
  - the manifest `files[].sha256`, or
  - the canonical published artifact reference for that file (for example, an HCS content reference) by resolving the artifact bytes and hashing them directly.
- If any manifest file path cannot be located at the commit, the signal MUST be `false`.

## Evidence (Recommended)

Implementations SHOULD preserve the repository URL, commit, and mismatch summary, for example:

```json
{
  "repo": "https://github.com/example-labs/pdf-processing-skill",
  "commit": "9fceb02e5f0f1a0b7b6c1b2d3e4f5a6b7c8d9e0f",
  "filesChecked": 12,
  "mismatches": [
    { "path": "scripts/extract.py", "reason": "sha_mismatch" }
  ],
  "checkedAt": "2026-03-02T12:00:00.000Z"
}
```
