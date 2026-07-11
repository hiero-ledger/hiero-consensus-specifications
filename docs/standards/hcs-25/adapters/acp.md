# HCS-25 (Adapter): ACP / Virtuals Performance (Informative)

## Purpose

Score marketplace-native execution performance (job success, volume, revenue, ratings) for ACP/Virtuals-style agents.

## Contribution

- Adapter id: `acp`
- Contribution mode: `scoped`
- Suggested weight: `2`

## Inputs (reference)

Uses marketplace-native metrics stored under `subject.metadata.metrics` and related fields.

See `../signals/acp.md`.

The reference adapter may apply percentile/baseline caps when converting “volume” and “revenue” to avoid outlier dominance.

## Output components (reference)

- `acp.jobs.successRate`
- `acp.jobs.deliveryVolume`
- `acp.jobs.revenue`
- `acp.reviews.rating`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:4QT8pTJ8gfsadCHRzSNSXT84oNT4QCiS648c1k9aRRUyXdhz69FYHD4QKEkmvH2Fyy;uid=0x0FFE28c5aBF64c6cD71c14d44575F3433c0A25e1;registry=virtuals-protocol;proto=acp;nativeId=0x0FFE28c5aBF64c6cD71c14d44575F3433c0A25e1`
