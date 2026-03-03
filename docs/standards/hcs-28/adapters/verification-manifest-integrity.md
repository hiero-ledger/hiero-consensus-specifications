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

## Semantics

`signals.manifestIntegrity.ok` MUST be `true` only when all of the following hold:

1. The verifier can retrieve and parse the HCS-26 `SKILL.json` manifest for the subject version.
2. The manifest satisfies HCS-26 validity requirements (including `SKILL.md` at the root and normalized paths without `..`).
3. For each entry in `SKILL.json.files[]`, the verifier can retrieve the referenced file content from its `hrl` and the computed `sha256` matches the manifest `files[].sha256`.

If any required manifest content is missing, malformed, or has a hash mismatch, `signals.manifestIntegrity.ok` MUST be `false`.

Hashing rules (baseline):

- `sha256` MUST be computed over the raw bytes of the resolved file content.
- `sha256` MUST be represented as lowercase hex in evidence and comparisons.

## Evidence (Recommended)

Implementations SHOULD preserve the manifest reference and mismatch summary, for example:

```json
{
  "manifestHrl": "hcs://1/0.0.22222",
  "filesChecked": 12,
  "missingFiles": [],
  "shaMismatches": [],
  "checkedAt": "2026-03-02T12:00:00.000Z"
}
```
