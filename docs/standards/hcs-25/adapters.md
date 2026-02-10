# HCS-25 (Appendix): Adapter Catalog (Informative)

This document complements [HCS-25](../hcs-25.md) by describing adapter concepts and providing pointers to the per-adapter catalog.

It is informative-only: implementations are free to define their own adapters, weights, and applicability rules.

Prefer the per-adapter catalog under [./adapters/index.md](./adapters/index.md) for the current, maintainable structure.

Adapters consume one or more trust signals and emit one or more normalized components. A single signal MAY inform multiple adapters, and an adapter MAY depend on multiple signals.

For signal schemas, see [Trust Signal Catalog](./signals.md) or the per-signal catalog under [./signals/index.md](./signals/index.md).

## Adapter identifier namespace pattern (normative)

Conforming implementations MUST use stable, namespaced adapter identifiers. HCS-25 defines this pattern; see [Adapter Identifier Namespacing](../hcs-25.md#adapter-identifier-namespacing).

## Adapter design checklist

An adapter SHOULD document:

- applicability rules (registry/protocol/class include/exclude)
- contribution mode (`universal` / `scoped` / `conditional`)
- component keys emitted
- normalization method for each component
- default value behavior (especially when signals are missing)

## Example adapter families (informative)

### Availability adapter

Purpose: reward reliably reachable endpoints and penalize long “offline” windows.

Typical inputs:

- `availabilityScore` ratio `[0,1]` or a last-seen timestamp

Example normalization:

- if a direct probe-based `availabilityScore` exists: `uptime = clamp(availabilityScore, 0, 1) * 100`
- else if `minsFromLastOnline` exists: linear decay over 24h:

```
uptime = 100 * clamp(1 - minsFromLastOnline / 1440, 0, 1)
```

Contribution mode guidance:

- `universal` for most endpoints where availability is always meaningful
- registry/protocol exclusions for entries that are not runtime endpoints (e.g., model catalogs)

### SimpleMath / SimpleScience adapter

Purpose: ensure baseline correctness and instruction compliance.

Inputs:

- SimpleMath signal record (see `./signals/simple-evals.md`)
- SimpleScience signal record (see `./signals/simple-evals.md`)

Normalization:

- `score` is typically binary (`0` or `100`) based on correctness; missing/timeout/unparseable are `0`.

Contribution mode guidance:

- `scoped` for protocols where interactive messaging is expected (e.g., chat-capable agents), so missing evals default to `0` instead of being excluded from the denominator.

### Feedback adapter (rating + volume)

Purpose: incorporate user feedback without letting a single rating dominate.

Inputs:

- `averageScore` in `[0,100]`
- `totalFeedbacks` (non-negative integer)

Example normalization pattern:

- `rating` component: `rating = averageScore` (clamped)
- `volume` component: log-scaled to avoid over-rewarding spam:

```
volumeRaw = min(volumeCap, log10(totalFeedbacks + 1) * volumeWeight)
volume = 100 * clamp(volumeRaw / volumeCap, 0, 1)
```

Contribution mode guidance:

- `scoped` where feedback is a first-class feature of the ecosystem; otherwise `conditional`.

### OSS popularity adapter (GitHub stars + downloads)

Purpose: incorporate adoption and ecosystem maturity signals for open-source components.

Inputs:

- `githubStars` and optionally `npmDownloads30d` / `pypiDownloads30d`

Example normalization:

Use log scaling and a weighted blend:

```
starsNorm = logScale(githubStars, starsCap)
downloadsNorm = logScale(downloadCount, downloadsCap)
score = 100 * (starsWeight * starsNorm + downloadsWeight * downloadsNorm)
```

Contribution mode guidance:

- `scoped` for software artifacts where OSS adoption is meaningful (e.g., MCP servers or SDK packages).

### Model benchmark adapters (leaderboards, preference scores)

Purpose: incorporate independent benchmark or preference signals for models.

Inputs:

- normalized `[0,100]` score plus sample size / coverage indicators

Normalization guidance:

- if a provider exposes unbounded raw scores (e.g., Elo-like), convert to `[0,100]` via rank/percentile or sigmoid scaling.
- use sample size/coverage to compute confidence and optionally down-weight low-coverage scores (see HCS-25 confidence section).

### Marketplace-specific adapters (ACP, AgentVerse, etc.)

Purpose: incorporate ecosystem-native trust signals (jobs delivered, verifier stats, response time, etc.).

Guidance:

- Publish the signal record format and the mapping to components so consumers can reason about the composite score.
- Avoid mixing unrelated signal sources into a single adapter when the sources have different gaming profiles; prefer separate adapters with explicit weights.

#### `model-tier`

- **Inputs**: subject model identifier plus signals indicating benchmark coverage (e.g., OpenRouter benchmark coverage and presence of other external eval statuses).
- **Outputs**: `model-tier.score` as a heuristic tier score.
- **Notes**: Intended as a fallback only when benchmarks are low coverage and no other external evals exist.
