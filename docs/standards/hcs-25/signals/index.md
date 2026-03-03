# HCS-25 (Appendix): Signal Catalog Index (Informative)

This index complements [HCS-25](../../hcs-25.md). It lists signal documents for ecosystems and providers currently documented under HCS-25.

## Overview

- Signal catalogs describe **what is collected** and **where it is stored** (example field schemas).
- Scoring is defined by trust score adapters (see [adapter catalog](../adapters/index.md)).

## Ecosystem mapping (Informative)

The following high-level mapping is an example of how an implementation might document common signals per ecosystem. Here “ecosystem” aligns with the `registry` routing parameter described in HCS-14.

| Ecosystem / registry | Common signal documents |
| --- | --- |
| `virtuals-protocol` (ACP) | [acp](./acp.md) |
| `erc-8004` | [erc8004-feedback](./erc8004-feedback.md), [ethos](./ethos.md) |
| `agentverse` / `uagent` | [agentverse-insights](./agentverse-insights.md), [agentverse-simple-evals](./agentverse-simple-evals.md) |
| `openrouter` / `near-ai` | [openrouter-evals](./openrouter-evals.md), [chatbot-arena](./chatbot-arena.md), [huggingface-model-index](./huggingface-model-index.md), [openllm-leaderboard](./openllm-leaderboard.md) |
| `mcp` (various registries) | [availability](./availability.md), [oss-popularity](./oss-popularity.md), [a2a-simple-evals](./a2a-simple-evals.md) |

## Signal docs (informative catalog)

The following pages describe example signal schemas. They are informative-only.

- [Availability / reachability signals](./availability.md)
- [Ethos account reputation (composite + per-source signals)](./ethos.md)
- [x402 onchain usage summary + cursor/checkpointing](./x402.md)
- [GitHub stars + NPM/PyPI downloads](./oss-popularity.md)
- [ACP / Virtuals marketplace-native performance metrics](./acp.md)
- [ERC-8004 feedback summary (rating + volume)](./erc8004-feedback.md)
- [OpenRouter category-ranking benchmarks (coverage-aware)](./openrouter-evals.md)
- [OpenLM Chatbot Arena signals (normalized score + raw Elo-like value)](./chatbot-arena.md)
- [Hugging Face model-index + popularity signals](./huggingface-model-index.md)
- [Open LLM leaderboard signals](./openllm-leaderboard.md)
- [AgentVerse “insights” signals](./agentverse-insights.md)
- [A2A/MCP SimpleMath/SimpleScience eval signals](./a2a-simple-evals.md)
- [AgentVerse/uAgent SimpleMath/SimpleScience eval signals](./agentverse-simple-evals.md)
- [NANDA SimpleMath/SimpleScience eval signals](./nanda-simple-evals.md)
- [SimpleMath / SimpleScience eval methodology (signal family)](./simple-evals.md)
