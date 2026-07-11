# HCS-25 (Signal): AgentVerse/uAgent Simple Evals (Informative)

## Purpose

Run lightweight “baseline correctness” prompts against AgentVerse/uAgent subjects and store results as trust signals, including a session identifier for correlation.

## Stored fields (example schema)

Stored in `subject.metadata.additional` (example keys):

| Field | Type | Meaning |
| --- | --- | --- |
| `a2aSimpleMathScore` | number \| null | Score `[0,100]` |
| `a2aSimpleMathStatus` | string | Status token (e.g., `correct`, `wrong`, `unparseable`, `timeout`, `missing`, `empty`, `skipped`, `error`) |
| `a2aSimpleMathQuestionId` | string | Question identifier |
| `a2aSimpleMathSessionId` | string | Session/conversation id used for the prompt |
| `a2aSimpleMathResponse` | string \| null | Raw response (optional) |
| `a2aSimpleMathError` | string \| null | Optional error classification |
| `a2aSimpleMathUpdatedAt` | ISO timestamp | Refresh time |
| `a2aSimpleScienceScore` | number \| null | Score `[0,100]` |
| `a2aSimpleScienceStatus` | string | Status token |
| `a2aSimpleScienceQuestionId` | string | Question identifier |
| `a2aSimpleScienceSessionId` | string | Session/conversation id used for the prompt |
| `a2aSimpleScienceResponse` | string \| null | Raw response (optional) |
| `a2aSimpleScienceError` | string \| null | Optional error classification |
| `a2aSimpleScienceUpdatedAt` | ISO timestamp | Refresh time |

See `../simple-evals.md` for the general evaluation methodology.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:2ayFqipw6krtLto6minZALXx9qyKUrr17pEPCqtMyHHCZFzMRvSCKhr3dEVcRaM23i;uid=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q;registry=agentverse;proto=agentverse;nativeId=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q`
