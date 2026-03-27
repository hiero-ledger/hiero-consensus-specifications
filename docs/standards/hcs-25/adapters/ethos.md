# HCS-25 (Adapter): Ethos Credibility (Informative)

## Purpose

Convert a provider-specific reputation score (Ethos) into a bounded trust contribution.

## Contribution

- Adapter id: `ethos`
- Contribution mode: `universal`
- Suggested weight: `1`
- Typical applicability: ecosystems where a stable identity can be mapped into the reputation system
- Typical exclusions: ecosystems where identity mapping is not defined or is expected to be missing (e.g., model catalogs like `openrouter`/`near-ai`, and mailbox-based ecosystems like `agentverse`/`uagent`)

## Inputs (reference)

Signal fields are stored in `subject.metadata` (see `../signals/ethos.md`), notably:

- `ethosComposite.score` (preferred) or `ethosScore`

## Output components

- `ethos.score` in `[0,100]`

## Normalization (reference)

The reference adapter maps a raw Ethos score into `[0,100]` using:

- a configured baseline and max (`baselineScore`, `maxScore`), and
- a configured maximum contribution budget (`maxContribution`).

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:4qS5GF2qowqjepLFNJx4jqvCHGuJ2ejHw6hXWuxpr19bKrC7hH4JC1o1dXFnjARGbr`
