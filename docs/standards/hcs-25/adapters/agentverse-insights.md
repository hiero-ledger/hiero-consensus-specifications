# HCS-25 (Adapter): AgentVerse Insights (Informative)

## Purpose

Convert AgentVerse “insights” indicators (rating and derived quality proxies) into a single normalized component.

## Contribution

- Adapter id: `agentverse-insights`
- Contribution mode: `scoped`
- Suggested weight: `1`

## Inputs (reference)

Stored under `subject.metadata.additional` (see `../signals/agentverse-insights.md`), including:

- `agentverseInsightsRating` (optional)
- readme/interactions/response-time/verifier counters (used as derived fallback inputs)

## Output components

- `agentverse-insights.score` in `[0,100]`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:2ayFqipw6krtLto6minZALXx9qyKUrr17pEPCqtMyHHCZFzMRvSCKhr3dEVcRaM23i;uid=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q;registry=agentverse;proto=agentverse;nativeId=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q`
