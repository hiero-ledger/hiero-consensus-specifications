# HCS-25 (Signal): Hugging Face Model Index (Informative)

## Purpose

Collect model metadata and evaluation-like indicators from Hugging Face, including a normalized score derived from:

- model-index metrics (when present), and/or
- popularity proxies (downloads/likes).

## Stored fields (example schema)

Stored in `subject.metadata.additional`:

| Field | Type | Meaning |
| --- | --- | --- |
| `huggingFaceModelId` | string \| null | Canonical HF model id |
| `openrouterHuggingFaceId` | string \| null | HF id resolved via OpenRouter |
| `huggingFaceEvalScore` | number \| null | Normalized score `[0,100]` |
| `huggingFaceSignalScore` | number \| null | Optional alias of `huggingFaceEvalScore` used by some ingest pipelines |
| `huggingFaceEvalMetricsCount` | number | Included metrics count |
| `huggingFaceDownloads` | number \| null | Popularity proxy |
| `huggingFaceLikes` | number \| null | Popularity proxy |
| `huggingFaceEvalMode` | `model-index` \| `popularity` \| `mixed` \| `missing` | Score mode |
| `huggingFaceEvalStatus` | `ok` \| `missing` \| `error` | Status |
| `huggingFaceEvalUpdatedAt` | ISO timestamp | Refresh time |
| `huggingFaceEvalSources` | string[] | Source identifiers |

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:82BHmpsiwqfeUTsazhWGkxj8UaV1q7qwVcXBZMS7PfDGkibnteEajdgCsd5qQYAm4f`
