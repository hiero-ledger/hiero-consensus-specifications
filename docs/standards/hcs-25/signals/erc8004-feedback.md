# HCS-25 (Signal): ERC-8004 Feedback Summary (Informative)

## Purpose

Collect ERC-8004 feedback summaries (average rating + volume) for agents registered under ERC-8004 networks.

## Applicability

Applied to ERC-8004 agents. The reference supports:

- EVM ERC-8004 feedback summaries, and
- optional Solana feedback ingestion (devnet).

## Stored fields (example schema)

Stored in `subject.metadata.erc8004FeedbackSummary`:

| Field | Type | Meaning |
| --- | --- | --- |
| `averageScore` | number | Average score `[0,100]` |
| `totalFeedbacks` | number | Feedback count (non-negative integer) |
| `registry` | string | Registry identifier |
| `network` | string | Network identifier |
| `updatedAt` | ISO timestamp | Refresh time |

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:4H2Rnf4oSM7N6HbFy8Su9qhUSpaE8yyVXoTj8UhXQ2qKfjQQ2CFjutJbJWy8NXKWi9;uid=11155111:724;registry=erc-8004;proto=erc-8004;nativeId=11155111:724`
