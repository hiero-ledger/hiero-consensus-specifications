# HCS-28 (Adapter): `safety.cisco-scan`

## Purpose

Score package safety using Cisco skill scan findings.

## Contribution

- Adapter id: `safety.cisco-scan`
- Contribution mode: `universal`
- Default weight: `1.00`
- Output key: `safety.cisco-scan.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- scanner execution status;
- vulnerability counts by severity (`critical`, `high`, `medium`, `low`).

Severity counts MUST be non-negative integers. If the scanner emits findings that do not map cleanly to these buckets, implementations MUST map them deterministically and SHOULD map unknown severities to `medium`.

## Normalization

If scan execution fails or no result exists in a universal contribution context, `score = 0`.

Otherwise:

```
raw = 100 - (30 * critical + 12 * high + 4 * medium + 1 * low)
score = clamp(round2(raw), 0, 100)
```

Implementations MAY use an equivalent deterministic mapping if documented in the scoring configuration version.

Implementations MUST emit `safety.cisco-scan.score` as a finite value in `[0,100]`.
