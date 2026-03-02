# HCS-28 (Adapter): `repository.health`

## Purpose

Estimate repository trustworthiness while resisting direct metric gaming.

## Contribution

- Adapter id: `repository.health`
- Contribution mode: `conditional`
- Default weight: `1.00`
- Output key: `repository.health.score`

## Applicability

Applies when external lookup is enabled and the skill points to a valid public source repository.

## Inputs

- repository metadata and activity signals;
- anti-abuse signals;
- freshness timestamps.

## Normalization

This adapter MUST expose only `repository.health.score` as its externally weighted score.

Implementations MUST combine internal repository signals with anti-gaming controls (for example, temporal decay, anomaly penalties, and smoothing) and clamp the result to `[0,100]`.

Implementations MUST NOT expose raw, directly gameable repository sub-metrics as separate weighted trust adapters in the HCS-28 baseline profile.

Implementations MUST emit `repository.health.score` as a finite value in `[0,100]`.
