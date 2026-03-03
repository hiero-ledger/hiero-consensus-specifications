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

The scan target MUST be derived deterministically from the HCS-26 skill subject. At minimum, implementations MUST include the exact files referenced by `SKILL.json.files[]` when constructing the scan input.

## Normalization

If scan execution fails or no result exists in a universal contribution context, `score = 0`.

Otherwise:

```
raw = 100 - (30 * critical + 12 * high + 4 * medium + 1 * low)
score = clamp(round2(raw), 0, 100)
```

Implementations MAY use an equivalent deterministic mapping if documented in the scoring configuration version.

Implementations MUST emit `safety.cisco-scan.score` as a finite value in `[0,100]`.

## Freshness (Baseline)

In baseline interoperability mode, persisted scan results SHOULD expire after 30 days. If an expired result is the only available result in read mode, the adapter MUST behave as if no result exists (which yields `0` in a universal contribution context).

## Evidence (Recommended)

Implementations SHOULD persist the severity bucket counts and scan timestamp used for the score, for example:

```json
{
  "critical": 0,
  "high": 1,
  "medium": 3,
  "low": 10,
  "computedAt": "2026-03-02T12:00:00.000Z"
}
```
