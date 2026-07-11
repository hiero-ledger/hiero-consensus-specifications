# HCS-25 (Signal): ACP / Virtuals Marketplace Metrics (Informative)

## Purpose

Describe marketplace-native “recent performance” metrics for ACP/Virtuals-style agents that can be transformed into normalized trust components.

## Storage (example)

Signals are stored on the subject record under `subject.metadata.metrics` (preferred) with fallbacks to a small number of top-level `subject.metadata.*` fields.

## Schema (example; non-exhaustive)

`subject.metadata.metrics` MAY include:

- `successRate` (number): success ratio as `0..1` or `0..100`.
- `successfulJobCount` (number): count of successful jobs in the provider’s window.
- `totalJobCount` (number): total jobs in the provider’s window.
- `volume` (number): provider-defined “delivery volume” in the provider’s window.
- `revenue` (number): provider-defined revenue in the provider’s window.
- `rating` (number): average rating in `0..5`.

Fallback fields (when `metadata.metrics` is absent) MAY include:

- `subject.metadata.successRate`
- `subject.metadata.successfulJobCount`

## Stored fields (example schema)

Stored in `subject.metadata.metrics` (preferred) or `subject.metadata.*` fallbacks.

| Field | Type | Meaning |
| --- | --- | --- |
| `metrics.successRate` | number \| null | Success ratio as `0..1` or `0..100` |
| `metrics.successfulJobCount` | number \| null | Successful job count |
| `metrics.totalJobCount` | number \| null | Total job count |
| `metrics.volume` | number \| null | Provider-defined “delivery volume” |
| `metrics.revenue` | number \| null | Provider-defined revenue |
| `metrics.rating` | number \| null | Average rating in `0..5` |
| `successRate` | number \| null | Fallback success rate when `metrics` is absent |
| `successfulJobCount` | number \| null | Fallback successful job count |

## Notes

- Implementations SHOULD treat these values as provider-scoped and SHOULD include provenance indicating the provider window (e.g., 7d/30d).
- Implementations often use baseline/percentile caps for volume and revenue when available to reduce outlier dominance.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:4QT8pTJ8gfsadCHRzSNSXT84oNT4QCiS648c1k9aRRUyXdhz69FYHD4QKEkmvH2Fyy;uid=0x0FFE28c5aBF64c6cD71c14d44575F3433c0A25e1;registry=virtuals-protocol;proto=acp;nativeId=0x0FFE28c5aBF64c6cD71c14d44575F3433c0A25e1`
