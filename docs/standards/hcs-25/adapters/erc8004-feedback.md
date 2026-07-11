# HCS-25 (Adapter): ERC-8004 Feedback (Informative)

## Purpose

Convert an ERC-8004 feedback summary (average rating + feedback volume) into trust components.

## Contribution

- Adapter id: `erc8004-feedback`
- Contribution mode: `scoped`
- Suggested weight: `1`

## Inputs (reference)

- `subject.metadata.erc8004FeedbackSummary` (see `../signals/erc8004-feedback.md`)

## Output components (reference)

- `erc8004-feedback.rating`
- `erc8004-feedback.volume` (log-scaled)

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:4H2Rnf4oSM7N6HbFy8Su9qhUSpaE8yyVXoTj8UhXQ2qKfjQQ2CFjutJbJWy8NXKWi9;uid=11155111:724;registry=erc-8004;proto=erc-8004;nativeId=11155111:724`
