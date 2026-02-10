# HCS-25 (Adapter): Connectivity Probe (Informative)

## Purpose

Probe a protocol adapter to determine whether the integration can establish a connection and get a non-empty acknowledgement.

## Contribution (reference)

- Adapter id: `connectivity`
- Contribution mode: `conditional`
- Suggested weight: `1`
- Typical applicability: ecosystems where a deterministic connectivity probe can be defined and audited
- Typical exclusions: model catalogs where “connectivity” is not meaningful (e.g., `openrouter`, `near-ai`)

## Inputs

Connectivity probes are ecosystem- and protocol-specific. A connectivity adapter SHOULD document:

- the target endpoint(s) or mailbox route(s) being tested,
- the exact probe request payload(s) used (including any required headers),
- a fixed time budget and retry policy, and
- how probe failures are classified (timeout vs error vs missing).

## Notes (reference)

This adapter is implementation-specific (it depends on a runtime adapter registry and connectivity probe semantics). Ecosystems SHOULD document probe messages and time budgets if they choose to expose a connectivity score.

## Output components

- Implementations MAY expose a single `connectivity.score` component in `[0,100]`.
- Implementations MAY instead expose one component per probe target, e.g. `connectivity.<target>` in `[0,100]`, where `<target>` is an implementation-defined identifier for the probed ecosystem/protocol.

## Production example (Registry Broker; informative)

The Registry Broker may expose `connectivity.score` for some ecosystems, but it is optional and may not be populated for all subjects. When present, it appears under `trustScores` on:

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
