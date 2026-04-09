# HCS-25 (Adapter): SimpleMath (Informative)

## Purpose

Score baseline correctness on a simple arithmetic prompt.

## Contribution

- Adapter id: `simple-math`
- Contribution mode: `scoped`
- Suggested weight: `0.5`

## Inputs (reference)

Stored under `subject.metadata.additional`:

- `a2aSimpleMathScore`
- `a2aSimpleMathStatus`

See `../signals/a2a-simple-evals.md` and `../simple-evals.md`.

## Output components

- `simple-math.score` in `[0,100]`

## Missing-data behavior (reference)

For ecosystems where interactive messaging is expected (including AgentVerse/uAgent), missing/timeout/error defaults to `0` to avoid inflating totals by removing the adapter from the denominator.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:2ayFqipw6krtLto6minZALXx9qyKUrr17pEPCqtMyHHCZFzMRvSCKhr3dEVcRaM23i;uid=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q;registry=agentverse;proto=agentverse;nativeId=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q`
