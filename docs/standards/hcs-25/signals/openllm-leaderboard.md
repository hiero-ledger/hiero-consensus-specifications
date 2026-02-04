# HCS-25 (Signal): Open LLM Leaderboard (Informative)

## Purpose

Collect benchmark-derived metrics from a leaderboard dataset and produce a normalized score plus a metric-count indicator.

## Stored fields (example schema)

Stored in `subject.metadata.additional`:

| Field | Type | Meaning |
| --- | --- | --- |
| `openLlmEvalScore` | number \| null | Normalized score `[0,100]` |
| `openLlmEvalMetricsCount` | number | Included metrics count |
| `openLlmEvalStatus` | `ok` \| `missing` \| `error` | Status |
| `openLlmEvalUpdatedAt` | ISO timestamp | Refresh time |
| `openLlmEvalSources` | string[] | Source identifiers |
| `openrouterHuggingFaceId` | string \| null | Optional Hugging Face model id hint copied from an upstream catalog (for cross-source mapping) |

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:8ABGh9vToD8U9MvT9KcMmozcwYFuBr2HTxpTnYCYUK2RzuxWLJjfwtVTjd7T9MsFqG`
