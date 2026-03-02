# HCS-28 (Adapter): `repository.health`

## Purpose

Estimate repository trustworthiness while resisting direct metric gaming.

## Contribution

- Adapter id: `repository.health`
- Contribution mode: `conditional`
- Default weight: `1.00`
- Output key: `repository.health.score`

## Applicability

Applies when the skill declares a valid public source repository URL.

If `includeExternal=false`, implementations MAY return a persisted score if available. If no persisted score exists and external I/O is disabled, implementations SHOULD return no score (so a conditional adapter does not contribute).

## Inputs

- repository URL (`repo`) and optional pinned commit (`commit`);
- repository metadata from a public host (for example, GitHub);
- freshness timestamp.

## Normalization

This adapter MUST expose only `repository.health.score` as its externally weighted score.

For baseline interoperability, implementations MUST compute `repository.health.score` deterministically from the following derived inputs when available:

- `isArchived` (boolean)
- `isFork` (boolean)
- `stars` (non-negative integer)
- `openIssues` (non-negative integer)
- `daysSinceLastPush` (non-negative integer)

If `isArchived=true`, `score` MUST be `0`.

Otherwise, compute:

1. `starsScore`:
   - `0` when `stars = 0`
   - `20` when `1 <= stars <= 9`
   - `40` when `10 <= stars <= 49`
   - `60` when `50 <= stars <= 199`
   - `80` when `200 <= stars <= 999`
   - `100` when `stars >= 1000`

2. `recencyScore`:
   - `100` when `daysSinceLastPush <= 7`
   - `85` when `<= 30`
   - `70` when `<= 90`
   - `55` when `<= 180`
   - `35` when `<= 365`
   - `15` when `> 365`

3. `issuesRatio = openIssues / max(1, stars)`.

4. `issuesScore`:
   - `100` when `issuesRatio <= 0.05`
   - `80` when `<= 0.20`
   - `60` when `<= 1.00`
   - `40` when `<= 5.00`
   - `20` when `> 5.00`

5. `forkPenalty = 20` when `isFork=true`, else `0`.

6. `raw = starsScore*0.45 + recencyScore*0.35 + issuesScore*0.20 - forkPenalty`.

7. `score = clamp(round2(raw), 0, 100)`.

Implementations MUST NOT expose raw, directly gameable repository sub-metrics as separate weighted trust adapters in the HCS-28 baseline profile.

Implementations MUST emit `repository.health.score` as a finite value in `[0,100]`.
