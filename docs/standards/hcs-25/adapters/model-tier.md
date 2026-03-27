# HCS-25 (Adapter): Model Tier (Fallback Heuristic) (Informative)

## Purpose

Provide a deterministic fallback score for model catalogs when:

- the primary benchmark signal is low-coverage, and
- no other independent eval sources are available.

## Contribution

- Adapter id: `model-tier`
- Contribution mode: `conditional`
- Suggested weight: `2`
- Typical applicability: model catalogs when external eval coverage is sparse

## Inputs (reference)

Reads model identifiers and benchmark-coverage status fields stored under `subject.metadata.additional` (e.g., OpenRouter coverage indicators and other eval statuses).

## Output components

- `model-tier.score` in `[0,100]`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:9TP2Y2ho4ATrYfALHooDFoH1iTdD4MrAKvxALkxYB2CcYjXuCTY9aXDzpMNCZGe6if`
