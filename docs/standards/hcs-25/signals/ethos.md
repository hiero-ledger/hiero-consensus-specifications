# HCS-25 (Signal): Ethos Reputation (Informative)

## Purpose

Collect reputation signals from Ethos for identities that can be represented as a stable Ethos “userkey” (for example, an EVM address or a linked X/Twitter handle).

## Applicability and exclusions

- Intended for EVM-style identities.
- Excludes ecosystems where Ethos coverage is structurally unavailable (e.g., AgentVerse/uAgent).

## Stored fields (example schema)

Stored in `subject.metadata`:

| Field | Type | Meaning |
| --- | --- | --- |
| `ethosUserkey` | string | Primary userkey used for the composite (e.g., `address:0x...`) |
| `ethosScore` | number \| null | Composite raw Ethos score (provider scale) |
| `ethosScoreStatus` | `ok` \| `missing` \| `error` | Composite status |
| `ethosScoreUpdatedAt` | ISO timestamp | Composite refresh time |
| `ethosSources` | array | Per-source signals used for the composite |
| `ethosComposite` | object | `{ version: 1, score, updatedAt, weights }` |

`ethosSources[]` entries:

| Field | Type | Meaning |
| --- | --- | --- |
| `userkey` | string | Source userkey |
| `kind` | `explicit` \| `x` \| `address` | Derivation type |
| `weight` | number | Source weight used in composite |
| `status` | `ok` \| `missing` \| `error` | Per-source status |
| `score` | number \| null | Per-source raw Ethos score |
| `updatedAt` | ISO timestamp | Per-source refresh time |

## Composite semantics

- The composite score is a weighted mean over sources that have a numeric score.
- Sources in `missing` state do not zero out the composite; they remain visible for explainability.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:4qS5GF2qowqjepLFNJx4jqvCHGuJ2ejHw6hXWuxpr19bKrC7hH4JC1o1dXFnjARGbr`
