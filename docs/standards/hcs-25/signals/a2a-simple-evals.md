# HCS-25 (Signal): A2A/MCP Simple Evals (Informative)

## Purpose

Run lightweight, automatically-graded “baseline correctness” prompts against chat-capable agents and store results as trust signals.

## Stored fields (example schema)

Stored in `subject.metadata.additional` (example keys):

| Field | Type | Meaning |
| --- | --- | --- |
| `a2aSimpleMathScore` | number \| null | Score `[0,100]` (typically 0/100) |
| `a2aSimpleMathStatus` | string | Status token (e.g., `correct`, `wrong`, `unparseable`, `timeout`, `missing`, `empty`, `skipped`, `upstream-error`, `error`) |
| `a2aSimpleMathQuestionId` | string | Question identifier |
| `a2aSimpleMathResponse` | string \| null | Raw response (optional) |
| `a2aSimpleMathError` | string \| null | Optional error classification (timeouts, upstream limits, etc.) |
| `a2aSimpleMathUpdatedAt` | ISO timestamp | Refresh time |
| `a2aSimpleScienceScore` | number \| null | Score `[0,100]` |
| `a2aSimpleScienceStatus` | string | Status token |
| `a2aSimpleScienceQuestionId` | string | Question identifier |
| `a2aSimpleScienceResponse` | string \| null | Raw response (optional) |
| `a2aSimpleScienceError` | string \| null | Optional error classification |
| `a2aSimpleScienceUpdatedAt` | ISO timestamp | Refresh time |

See `../simple-evals.md` for the general evaluation methodology.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:3RomW1LwBJ7ZM1PWrLCEro9w4YtY9xwGNWMM2i21mANv8BcWpKg4a7zXxcoNMPDJ7B` (NANDA registry entry with simple eval fields populated)
