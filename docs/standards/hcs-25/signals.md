# HCS-25 (Appendix): Trust Signal Catalog (Informative)

This document complements [HCS-25](../hcs-25.md) by describing trust signal concepts and providing pointers to the per-signal catalog.

It is informative-only: the exact signals and providers are expected to vary by ecosystem.

Prefer the per-signal catalog under [./signals/index.md](./signals/index.md) for the current, maintainable structure.

Quick links:

- [Signal catalog (index)](./signals/index.md)
- [SimpleMath / SimpleScience eval methodology (signal family)](./signals/simple-evals.md)

## Signal record shape (recommended)

A signal record SHOULD be represented as a small object that includes:

- `status`: `ok` | `missing` | `timeout` | `error` | `stale`
- `fetchedAt`: an ISO timestamp indicating when the signal was last refreshed
- `sources`: a list of stable source identifiers (strings) describing where the signal was derived from
- `data`: a provider-neutral payload containing the values needed by scoring adapters

Example (illustrative):

```json
{
  "status": "ok",
  "fetchedAt": "2026-01-14T00:00:00Z",
  "sources": ["example:monitoring:v1"],
  "data": {
    "uptime_30d": 0.9981,
    "p50_latency_ms": 210
  }
}
```

## Signal identifier namespace pattern (normative)

Conforming implementations MUST use stable, namespaced signal identifiers. HCS-25 defines a dot-separated namespace convention; see [Signal Identifier Namespacing](../hcs-25.md#signal-identifier-namespacing).

In short:

```
{namespace}.{name}[.{subname}...]
```

Where each segment is lowercase ASCII and matches `/[a-z][a-z0-9_-]*/`.

Implementations MUST document their chosen signal IDs and how they map to adapters.

## Per-signal catalog (informative)

The maintainable catalog lives under [./signals/index.md](./signals/index.md) and includes per-signal schema pages (examples) for:

- marketplace ecosystems (ACP, AgentVerse, ERC-8004),
- operational signals (availability),
- adoption signals (OSS popularity),
- usage signals (x402), and
- model eval signals (OpenRouter, Chatbot Arena, Hugging Face, Open LLM Leaderboard).

For adapter mappings, see [Adapter Catalog](./adapters.md) or [./adapters/index.md](./adapters/index.md).

Stored in: subject `metadata.additional`.

| Field | Type | Meaning |
| --- | --- | --- |
| `chatbotArenaEvalScore` | number \| null | Normalized score `[0,100]` computed from min/max Elo observed in the snapshot |
| `chatbotArenaEvalElo` | number \| null | Raw Elo-like value from the source table |
| `chatbotArenaEvalVotes` | number \| null | Vote/sample size indicator (if present) |
| `chatbotArenaEvalStatus` | `ok` \| `missing` | Availability status |
| `chatbotArenaEvalUpdatedAt` | ISO timestamp | Refresh time |
| `chatbotArenaEvalSources` | string[] | Sources list (e.g., `chatbot-arena:openlm`) |

### `huggingface-model-index` (model metadata + eval-like aggregation)

Stored in: subject `metadata.additional`.

| Field | Type | Meaning |
| --- | --- | --- |
| `huggingFaceModelId` | string \| null | Canonical HF model id |
| `openrouterHuggingFaceId` | string \| null | HF id resolved via OpenRouter model list |
| `huggingFaceEvalScore` | number \| null | Normalized score `[0,100]` derived from model-index metrics and/or popularity |
| `huggingFaceEvalMetricsCount` | number | Count of included metrics |
| `huggingFaceDownloads` | number \| null | Popularity proxy |
| `huggingFaceLikes` | number \| null | Popularity proxy |
| `huggingFaceEvalMode` | `model-index` \| `popularity` \| `mixed` \| `missing` | How the score was produced |
| `huggingFaceEvalStatus` | `ok` \| `missing` \| `error` | Availability status |
| `huggingFaceEvalUpdatedAt` | ISO timestamp | Refresh time |
| `huggingFaceEvalSources` | string[] | Sources list (e.g., `huggingface:model-index`, `huggingface:popularity`) |

### `openllm-leaderboard` (Open LLM Leaderboard dataset)

Stored in: subject `metadata.additional`.

| Field | Type | Meaning |
| --- | --- | --- |
| `openLlmEvalScore` | number \| null | Normalized score `[0,100]` derived from selected benchmark metrics |
| `openLlmEvalMetricsCount` | number | Count of included metrics |
| `openLlmEvalStatus` | `ok` \| `missing` \| `error` | Availability status |
| `openLlmEvalUpdatedAt` | ISO timestamp | Refresh time |
| `openLlmEvalSources` | string[] | Sources list (e.g., `openllm:leaderboard:v2`) |
