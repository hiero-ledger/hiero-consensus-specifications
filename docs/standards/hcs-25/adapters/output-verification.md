# HCS-25 (Adapter): Output Verification (Informative)

## Purpose

Convert per-decision reasoning verification scores into trust components, providing a real-time output quality dimension to the composite AI Trust Score.

While existing adapters measure **historical performance** (benchmarks, ratings, uptime) or **adoption** (downloads, stars), this adapter measures **ongoing output correctness** — whether the agent's reasoning holds up under independent verification at decision time.

## Contribution

- Adapter id: `output-verification`
- Contribution mode: `scoped`
- Suggested weight: `1`
- Typical applicability: agents that route decisions through independent verification before settlement or execution
- Typical exclusions: model catalogs, static tool endpoints, agents without verification integration

## Inputs (reference)

- `subject.metadata.outputVerificationSummary` (see `../signals/output-verification.md`)

## Output components

- `output-verification.quality` in `[0,100]` — stake-weighted quality score
- `output-verification.coverage` in `[0,100]` — log-scaled verification volume

## Normalization (reference)

### `output-verification.quality`

Combines allow rate, calibrated confidence, and discriminative power:

```
// Base quality: how often does the agent pass verification with high confidence?
baseQuality = allowRate × avgConfidence

// Discriminative power: does the verifier actually reject bad outputs?
// A verifier that ALLOWs everything (blockRate ≈ 0) provides no signal.
discriminativePower = clamp(blockRate / 0.05, 0, 1)
// Full credit when blockRate ≥ 5% (verifier demonstrably rejects some outputs)

// Stake weighting (when stakeDistribution is available):
// High-stake checks count more than trivial ones.
// Default: equal weight if no stake data.
stakeMultiplier = 1.0
if stakeDistribution:
    totalWeighted = (low.checks × 0.5 + medium.checks × 1.0
                     + high.checks × 2.0 + critical.checks × 3.0)
    totalUnweighted = sum(all checks)
    stakeMultiplier = clamp(totalWeighted / totalUnweighted, 0.5, 3.0)

quality = round(baseQuality × discriminativePower × min(stakeMultiplier, 1.5) × 100)
quality = clamp(quality, 0, 100)
```

This normalization addresses three gaming vectors:

1. **Rubber-stamp verifiers**: `discriminativePower` penalizes verifiers that never BLOCK
2. **Trivial-claim farming**: `stakeMultiplier` weights high-stake checks more heavily
3. **Inflated confidence**: `avgConfidence` is calibrated — a provider claiming 0.99 on everything but getting 10% wrong will have low calibrated confidence

### `output-verification.coverage`

Log-scaled to reward consistent verification without over-rewarding volume:

```
volumeCap = 50  // configurable
volumeRaw = min(volumeCap, log10(totalChecks + 1) × 20)
coverage = 100 × clamp(volumeRaw / volumeCap, 0, 1)
```

Minimum threshold: `totalChecks >= 10` required; below this the adapter returns no components (insufficient sample).

## Adapter total

Weighted mean favoring quality:

```
total = 0.7 × quality + 0.3 × coverage
```

Implementations MAY adjust internal component weights.

## Applicability guidance

- **Include**: Any agent whose outputs are verified by one or more conforming verification providers
- **Exclude**: Agents with `totalChecks < 10` in the current window (insufficient sample size)
- **Exclude**: Self-reported verification (agent verifying its own output provides no trust signal)
- **Registry**: Applicable across all registries — output verification is chain-agnostic
- **Protocol**: Applicable to any protocol

## Multi-provider support

When multiple verification providers report on the same agent, implementations SHOULD:

1. Compute components per-provider
2. Weight by each provider's `checksContributed` (log-scaled)
3. Cross-reference: agreement between independent providers strengthens confidence; disagreement warrants lower scores

```
providerWeight_i = log10(provider_i.checksContributed + 1)
combined.quality = sum(providerWeight_i × provider_i.quality) / sum(providerWeight_i)
```

## Relationship to other adapters

| Adapter | What it measures | Temporal scope | Independence |
| --- | --- | --- | --- |
| `availability` | Is the agent online? | Current | Self/probe |
| `erc8004-feedback` | User satisfaction | Retrospective | User-reported |
| `chatbot-arena` | Model capability (ELO) | Retrospective | Third-party |
| `openllm-leaderboard` | Benchmark scores | Retrospective | Third-party |
| `simple-math` | Basic correctness | One-shot | Automated |
| **`output-verification`** | **Is THIS output correct?** | **Per-decision** | **Independent verifier** |

## Security considerations

- **Provider trust**: The adapter's value depends entirely on the verification provider's independence and methodology quality. Implementations SHOULD require providers to publish methodology documentation.
- **Collusion risk**: An agent and its verification provider could collude to inflate scores. Multi-provider support and cross-referencing mitigates this.
- **Circular trust**: Avoid configurations where the verification provider's own trust score depends on the agents it verifies.
- **Gaming via trivial claims**: The `discriminativePower` and `stakeMultiplier` components specifically resist this vector. Implementations SHOULD monitor for anomalous stake distributions.

## Reference implementations (informative)

| Provider | Method | Attestation |
| --- | --- | --- |
| ThoughtProof | Adversarial multi-model consensus with calibrated confidence | ERC-8004 Agent #28388, signer `0xAbDdE1A06eEBD934fea35D4385cF68F43aCc986d` on Base |

_Additional conforming providers will be listed as they implement the signal schema._

## License

Apache-2.0
