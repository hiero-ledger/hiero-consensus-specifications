# HCS-25 (Signal): AgentVerse Insights (Informative)

## Purpose

Collect marketplace-native “insights” signals for AgentVerse/uAgent subjects, including quality indicators and verifier interaction counters.

## Stored fields (example schema)

Stored in `subject.metadata.additional`:

| Field | Type | Meaning |
| --- | --- | --- |
| `agentverseInsightsUpdatedAt` | ISO timestamp | Refresh time |
| `agentverseInsightsSources` | string[] | Source identifiers |
| `agentverseInsightsStatus` | `ok` \| `missing` \| `upstream-error` | Status |
| `agentverseInsightsAddress` | string | AgentVerse address |
| `agentverseInsightsContract` | `mainnet` \| `testnet` \| null | Contract network |
| `agentverseInsightsRating` | number \| null | Optional rating (often `0–5`) |
| `agentverseInsightsReadmeQualityScore` | number \| null | Ratio `[0,1]` |
| `agentverseInsightsReadmeUniquenessScore` | number \| null | Ratio `[0,1]` |
| `agentverseInsightsInteractionsScore` | number \| null | Ratio `[0,1]` |
| `agentverseInsightsAvgResponseTime` | number \| null | Seconds |
| `agentverseInsightsAsi1TotalInteractions` | number | Total ASI-1 interactions |
| `agentverseInsightsAsi1TotalSuccessInteractions` | number | Total successful ASI-1 interactions |
| `agentverseInsightsAsi1RecentInteractions` | number | Recent ASI-1 interactions |
| `agentverseInsightsAsi1RecentSuccessInteractions` | number | Recent successful ASI-1 interactions |
| `agentverseInsightsVerifierTotalInteractions` | number | Total verifier interactions |
| `agentverseInsightsVerifierTotalSuccessInteractions` | number | Total verifier successes |
| `agentverseInsightsVerifierRecentInteractions` | number | Recent verifier interactions |
| `agentverseInsightsVerifierRecentSuccessInteractions` | number | Recent verifier successes |

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:2ayFqipw6krtLto6minZALXx9qyKUrr17pEPCqtMyHHCZFzMRvSCKhr3dEVcRaM23i;uid=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q;registry=agentverse;proto=agentverse;nativeId=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q`
