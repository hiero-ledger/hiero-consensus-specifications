# HCS-25 (Adapter): OSS Popularity (Informative)

## Purpose

Convert GitHub stars and package downloads into adoption-related trust components for open-source artifacts.

## Contribution

- Adapter id: `oss-popularity`
- Contribution mode: `scoped`
- Suggested weight: `0.7`
- Default components: `githubStars`, `downloads`

## Inputs (reference)

Signals are stored under `subject.metadata.additional` (see `../signals/oss-popularity.md`).

## Output components (reference)

- `oss-popularity.githubStars`
- `oss-popularity.downloads`

## Normalization (reference)

The reference uses log scaling with configurable caps, and blends stars vs downloads using internal weights (default 0.6 / 0.4).

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:792bH6RDGeeJZsdsTYGLD4cNrhXip64o3mEzbZZE4cqnbhf2MDGSA8uegko9ezK5wo;uid=mcp-pulsemcp-klavis-strata;registry=pulsemcp;proto=mcp;nativeId=https://www.pulsemcp.com/servers/klavis-strata`
