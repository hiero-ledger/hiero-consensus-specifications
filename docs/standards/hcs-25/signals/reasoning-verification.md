# HCS-25 (Signal): Reasoning Verification Summary (Informative)

## Purpose

Collect reasoning verification summaries for agents that route decisions through an adversarial multi-model verification service before settlement or execution.

## Applicability

Applied to agents with an active verification integration. The signal adapter queries the verification service for historical verdict data.

## Stored fields (example schema)

Stored in `subject.metadata.reasoningVerificationSummary`:

| Field | Type | Meaning |
| --- | --- | --- |
| `allowRate` | number | Ratio of ALLOW verdicts `[0,1]` over scoring window |
| `avgConfidence` | number | Mean calibrated confidence `[0,1]` across all checks |
| `blockRate` | number | Ratio of BLOCK verdicts `[0,1]` |
| `uncertainRate` | number | Ratio of UNCERTAIN verdicts `[0,1]` |
| `totalChecks` | number | Total verification checks in scoring window |
| `avgDurationMs` | number | Mean verification latency (ms) |
| `verifierAddress` | string | On-chain signer address for provenance |
| `scoringWindow` | string | Aggregation window (e.g., `7d`, `30d`) |
| `updatedAt` | ISO timestamp | Last refresh time |

## Signal provenance

- **Source type**: API query to verification service
- **Refresh**: on-demand or periodic (suggested: every 6 hours)
- **Verification**: the `verifierAddress` can be cross-checked against `GET /v1/signer` on the verification service, and individual attestations can be verified on-chain via `ecrecover()`

## Example signal source

ThoughtProof (`https://api.thoughtproof.ai`):

- `POST /v1/check` — per-decision verification with `onchain_proof`
- `GET /v1/signer` — returns the ECDSA signer address
- Verdict schema: `ALLOW | BLOCK | UNCERTAIN` with `confidence` (0–1) and `confidence_bps` (uint16)
