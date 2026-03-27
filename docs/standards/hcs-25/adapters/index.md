# HCS-25 (Appendix): Adapter Catalog Index (Informative)

This index complements [HCS-25](../../hcs-25.md). It lists adapter documents that describe how trust signals are mapped to normalized components and how they contribute to the composite trust score.

## Overview

- Adapter docs describe **what signals are used**, **what component keys are emitted**, **how normalization works**, and **default weights/scope** (when an implementation publishes them).
- Signal schemas are documented separately (see [signal catalog](../signals/index.md)).

## Ecosystem mapping (Informative)

The following high-level mapping is an example of how an implementation might document common adapters per ecosystem. Here “ecosystem” aligns with the `registry` routing parameter described in HCS-14.

| Ecosystem / registry | Common trust score adapters |
| --- | --- |
| `virtuals-protocol` (ACP) | [availability](./availability.md), [acp](./acp.md) |
| `erc-8004` | [availability](./availability.md), [erc8004-feedback](./erc8004-feedback.md), [ethos](./ethos.md) |
| `agentverse` / `uagent` | [availability](./availability.md), [agentverse-insights](./agentverse-insights.md), [agentverse-verifier](./agentverse-verifier.md), [simple-math](./simple-math.md), [simple-science](./simple-science.md) |
| `openrouter` / `near-ai` | [openrouter-evals](./openrouter-evals.md), [chatbot-arena](./chatbot-arena.md), [huggingface-model-index](./huggingface-model-index.md), [openllm-leaderboard](./openllm-leaderboard.md), [model-tier](./model-tier.md) |
| `mcp` (various registries) | [availability](./availability.md), [oss-popularity](./oss-popularity.md), [simple-math](./simple-math.md), [simple-science](./simple-science.md), [connectivity](./connectivity.md) |

## Trust score adapter docs (informative catalog)

The following pages describe example trust score adapters. They are informative-only.

- [Availability / uptime](./availability.md)
- [Ethos credibility contribution](./ethos.md)
- [ACP/Virtuals marketplace performance](./acp.md)
- [ERC-8004 feedback scoring (rating + volume)](./erc8004-feedback.md)
- [x402 usage scoring (volume + trades)](./x402.md)
- [OSS popularity scoring (stars + downloads)](./oss-popularity.md)
- [SimpleMath scoring](./simple-math.md)
- [SimpleScience scoring](./simple-science.md)
- [AgentVerse insights scoring](./agentverse-insights.md)
- [AgentVerse verifier reliability scoring](./agentverse-verifier.md)
- [OpenRouter benchmarks scoring](./openrouter-evals.md)
- [Chatbot Arena scoring](./chatbot-arena.md)
- [Hugging Face model index scoring](./huggingface-model-index.md)
- [Open LLM leaderboard scoring](./openllm-leaderboard.md)
- [Fallback model tier heuristic](./model-tier.md)
- [Optional connectivity probe scoring](./connectivity.md)
