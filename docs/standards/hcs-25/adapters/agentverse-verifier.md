# HCS-25 (Adapter): AgentVerse Verifier Reliability (Informative)

## Purpose

Convert verifier interaction counters (success rates + response time) into a reliability score, with a confidence factor based on interaction volume.

## Contribution

- Adapter id: `agentverse-verifier`
- Contribution mode: `conditional`
- Suggested weight: `1`

## Inputs (reference)

Stored under `subject.metadata.additional` (see `../signals/agentverse-insights.md`):

- total/recent verifier interactions + successes
- average response time

## Output components

- `agentverse-verifier.score` in `[0,100]`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:2ayFqipw6krtLto6minZALXx9qyKUrr17pEPCqtMyHHCZFzMRvSCKhr3dEVcRaM23i;uid=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q;registry=agentverse;proto=agentverse;nativeId=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q`
