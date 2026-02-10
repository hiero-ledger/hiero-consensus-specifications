# HCS-25 (Adapter): OpenRouter Benchmarks (Informative)

## Purpose

Convert OpenRouter category-ranking signals into a coverage-aware benchmark component.

## Contribution

- Adapter id: `openrouter-evals`
- Contribution mode: `scoped`
- Suggested weight: `0.5`
- Typical applicability: model catalogs (e.g., `openrouter`, `near-ai`)

## Inputs (reference)

See `../signals/openrouter-evals.md`.

## Output components

- `openrouter-evals.score` in `[0,100]`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:A4ESM82SuJswkaEjd2ntyMyhY6o3HKMuW2JzRFVg7dNGdmKSwtX2DpbptjW7YrnEvs`
