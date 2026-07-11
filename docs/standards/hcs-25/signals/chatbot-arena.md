# HCS-25 (Signal): Chatbot Arena (OpenLM) (Informative)

## Purpose

Collect preference-style “Arena” signals for models and provide both a normalized score and the raw Elo-like value.

## Stored fields (example schema)

Stored in `subject.metadata.additional`:

| Field | Type | Meaning |
| --- | --- | --- |
| `chatbotArenaEvalScore` | number \| null | Normalized score `[0,100]` derived from snapshot min/max |
| `chatbotArenaEvalElo` | number \| null | Raw Elo-like value from the source |
| `chatbotArenaEvalVotes` | number \| null | Vote/sample size indicator (if available) |
| `chatbotArenaEvalStatus` | `ok` \| `missing` | Status |
| `chatbotArenaEvalUpdatedAt` | ISO timestamp | Refresh time |
| `chatbotArenaEvalSources` | string[] | Source identifiers |

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:A4ESM82SuJswkaEjd2ntyMyhY6o3HKMuW2JzRFVg7dNGdmKSwtX2DpbptjW7YrnEvs`
