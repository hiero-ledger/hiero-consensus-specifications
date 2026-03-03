# HCS-25 (Adapter): Open LLM Leaderboard (Informative)

## Purpose

Expose a benchmark-derived leaderboard score as a normalized component for models.

## Contribution

- Adapter id: `openllm-leaderboard`
- Contribution mode: `conditional`
- Suggested weight: `1`
- Typical applicability: models where an Open LLM leaderboard record exists

## Inputs (reference)

See `../signals/openllm-leaderboard.md`.

## Output components

- `openllm-leaderboard.score` in `[0,100]`

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:8ABGh9vToD8U9MvT9KcMmozcwYFuBr2HTxpTnYCYUK2RzuxWLJjfwtVTjd7T9MsFqG`
