# HCS-25 (Adapter): Reasoning Verification (Informative)

## Purpose

Score whether an agent's outputs have been independently verified for reasoning quality through adversarial multi-model consensus. Unlike retrospective adapters (benchmarks, ratings), this adapter reflects **real-time, per-decision verification** — whether the agent's reasoning was checked and found sound at the point of action.

## Contribution

- Adapter id: `reasoning-verification`
- Contribution mode: `scoped`
- Suggested weight: `1`
- Typical applicability: agents that route decisions through a reasoning verification service before settlement or execution
- Typical exclusions: model catalogs, static tool endpoints, agents without verification integration

## Inputs (reference)

Uses `subject.metadata.reasoningVerificationSummary`:

| Field | Type | Meaning |
| --- | --- | --- |
| `allowRate` | number | Ratio of ALLOW verdicts `[0,1]` over scoring window |
| `avgConfidence` | number | Mean calibrated confidence `[0,1]` across all checks |
| `blockRate` | number | Ratio of BLOCK verdicts `[0,1]` |
| `uncertainRate` | number | Ratio of UNCERTAIN verdicts `[0,1]` |
| `totalChecks` | number | Total verification checks in scoring window (non-negative integer) |
| `avgDurationMs` | number | Mean verification latency in milliseconds |
| `verifierAddress` | string | On-chain signer address of the verification service (for provenance) |
| `scoringWindow` | string | Time window for aggregation (e.g., `7d`, `30d`) |
| `updatedAt` | ISO timestamp | Last refresh time |

## Output components

- `reasoning-verification.quality` in `[0,100]` — composite of allow rate × confidence
- `reasoning-verification.coverage` in `[0,100]` — log-scaled verification volume

## Normalization (reference)

### `reasoning-verification.quality`

Combines allow rate and calibrated confidence:

```
raw = allowRate × avgConfidence
quality = round(raw × 100)
```

A high quality score requires both: the agent's decisions are mostly approved (high allow rate) AND the verification service is confident in those approvals (high confidence). An agent with 100% allow rate but low confidence scores lower than one with 95% allow rate and high confidence.

### `reasoning-verification.coverage`

Log-scaled check volume against a configurable cap:

```
coverage = round(min(log1p(totalChecks) / log1p(cap), 1) × 100)
```

Reference cap: `1000` (an agent with 1000+ verified decisions in the scoring window scores 100).

## Relationship to other adapters

- **Availability** measures whether an agent is reachable. Reasoning verification measures whether its outputs are *correct*.
- **ERC-8004 feedback** captures post-hoc user ratings. Reasoning verification captures pre-settlement, automated quality checks.
- **Eval benchmarks** (simple-math, openllm-leaderboard) measure static capability. Reasoning verification measures live decision quality on real tasks.

This adapter is complementary to all of the above. An agent can have perfect uptime, high benchmark scores, and good ratings, but still produce flawed reasoning on a specific decision. Reasoning verification catches that.

## Security considerations

- The `verifierAddress` field enables consumers to validate that verification was performed by a known, trusted service (e.g., by checking the address against a registry of approved verifiers).
- Implementations SHOULD verify that the verification service uses adversarial multi-model consensus rather than single-model checking, as single-model verification is susceptible to correlated failures.
- The adapter does not mandate a specific verification protocol. Implementations MAY accept verification data from any service that produces the required signal schema.

## Example signal sources

- [ThoughtProof](https://api.thoughtproof.ai/v1/signer) — multi-model adversarial reasoning verification with on-chain ECDSA attestations (EIP-191). Returns ALLOW/BLOCK/UNCERTAIN verdicts with calibrated confidence scores.

## Production example (informative)

No production deployment of this adapter exists yet. A reference integration could query a verification service's historical verdicts for a given agent and compute the summary fields above.
