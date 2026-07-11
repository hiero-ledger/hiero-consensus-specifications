# HCS-25 (Signal): Output Verification Summary (Informative)

## Purpose

Collect per-decision reasoning verification results from independent verification services, providing real-time output quality signals for AI agents.

## Applicability

Applied to any AI agent whose outputs are independently verified through adversarial or multi-model verification pipelines. Applicable across registries and protocols — output verification is transport-agnostic.

## Background

Unlike retrospective trust signals (benchmarks, historical ratings, uptime), output verification operates **per-decision**: each agent output is independently checked for reasoning quality before settlement or downstream use.

This signal type answers "was THIS specific output correct?" rather than "how has this agent performed historically?"

## Collection method

Signal adapters query a verification provider's stats endpoint. The endpoint MUST return an aggregated verification summary for the agent over a configurable window.

### Reference endpoint format

```
GET {provider_base_url}/v1/stats/{agent_id}?window=7d
```

### Minimum provider requirements

A conforming verification provider MUST:

1. Perform independent assessment of agent outputs (not self-reported by the agent)
2. Return structured verdicts with calibrated confidence scores
3. Provide an aggregated stats endpoint with the fields defined below
4. Publish methodology documentation sufficient for reproducibility

A conforming verification provider SHOULD:

1. Use multiple independent models or evaluators (adversarial consensus)
2. Provide on-chain attestations for audit trail (e.g., via ERC-8004 or HCS-7)
3. Publish calibration methodology and ground-truth validation results
4. Make benchmark datasets publicly available

## Stored fields (example schema)

Stored in `subject.metadata.outputVerificationSummary`:

| Field | Type | Meaning |
| --- | --- | --- |
| `allowRate` | number | Ratio of outputs that passed verification `[0,1]` |
| `blockRate` | number | Ratio of outputs that failed verification `[0,1]` |
| `uncertainRate` | number | Ratio of outputs with inconclusive verification `[0,1]` |
| `avgConfidence` | number | Mean calibrated confidence across all checks `[0,1]` |
| `totalChecks` | number | Total verification count (non-negative integer) |
| `stakeDistribution` | object | Optional breakdown by stake level (see below) |
| `windowDays` | number | Lookback window in days |
| `providers` | array | List of verification provider records (see below) |
| `updatedAt` | ISO timestamp | Refresh time |

### Provider record

Each entry in `providers[]`:

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | string | Stable provider identifier |
| `signerAddress` | string | On-chain signer address (if applicable) |
| `methodologyUrl` | string | URL to verification methodology documentation |
| `checksContributed` | number | Number of checks from this provider |

### Stake distribution (optional)

When the verification service distinguishes claim difficulty or stake level, `stakeDistribution` MAY contain:

| Field | Type | Meaning |
| --- | --- | --- |
| `low` | object | `{ checks, allowRate, avgConfidence }` for low-stake claims |
| `medium` | object | Same structure for medium-stake claims |
| `high` | object | Same structure for high-stake claims |
| `critical` | object | Same structure for critical-stake claims |

This enables adapters to weight high-stake verification more heavily than trivial checks.

## Signal provenance

Each verification provider MUST be identified by:

- **Provider identifier**: A stable, unique string
- **Method**: Description of the verification approach
- **Attestation**: On-chain signer address or equivalent verifiable credential, if available

## Refresh cadence (informative)

Recommended: every 1–6 hours, depending on agent activity volume.

## Gaming resistance considerations

Verification providers SHOULD document how their pipeline resists gaming. Common resistance mechanisms include:

- **Multi-evaluator consensus**: Multiple independent models/evaluators must agree
- **Calibrated confidence**: Scores validated against ground truth, not raw model outputs
- **On-chain attestations**: Tamper-evident audit trail
- **Adversarial architecture**: Evaluators incentivized to find flaws, not rubber-stamp outputs
- **Stake-level weighting**: High-stake verifications contribute more than trivial checks

## Limitations

- Verification quality depends entirely on the provider's methodology and calibration
- No single verification provider should be treated as infallible (see HCS-25 §Interpretation Guidance)
- Implementations SHOULD consider supporting multiple verification providers and cross-referencing results
- An agent that only verifies trivial claims provides weaker signal than one verifying high-stake decisions

## License

Apache-2.0
