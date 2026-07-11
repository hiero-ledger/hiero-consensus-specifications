# HCS-25 (Signal): NANDA Simple Evals (Informative)

## Purpose

Run lightweight “baseline correctness” prompts against NANDA A2A endpoints and store results as trust signals, including error classification for upstream billing/limits.

## Stored fields (example schema)

Stored in `subject.metadata.additional` (example keys):

| Field | Type | Meaning |
| --- | --- | --- |
| `nandaSimpleMathScore` | number \| null | Score `[0,100]` |
| `nandaSimpleMathStatus` | string | Status token (e.g., `correct`, `wrong`, `unparseable`, `timeout`, `missing`, `empty`, `skipped`, `upstream-error`, `error`) |
| `nandaSimpleMathQuestionId` | string | Question identifier |
| `nandaSimpleMathResponse` | string \| null | Raw response (optional) |
| `nandaSimpleMathError` | string \| null | Optional error classification (timeouts, upstream limits, etc.) |
| `nandaSimpleMathUpdatedAt` | ISO timestamp | Refresh time |
| `nandaSimpleScienceScore` | number \| null | Score `[0,100]` |
| `nandaSimpleScienceStatus` | string | Status token |
| `nandaSimpleScienceQuestionId` | string | Question identifier |
| `nandaSimpleScienceResponse` | string \| null | Raw response (optional) |
| `nandaSimpleScienceError` | string \| null | Optional error classification |
| `nandaSimpleScienceUpdatedAt` | ISO timestamp | Refresh time |

See `../simple-evals.md` for the general evaluation methodology.

## Interop note (informative)

Some implementations normalize NANDA results into the shared `a2aSimple*` key family so that a single SimpleMath/SimpleScience scoring adapter can be reused across ecosystems. See `./a2a-simple-evals.md` for that shared schema.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:3RomW1LwBJ7ZM1PWrLCEro9w4YtY9xwGNWMM2i21mANv8BcWpKg4a7zXxcoNMPDJ7B`
