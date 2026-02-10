# HCS-25 (Signal): Availability / Reachability (Informative)

## Purpose

Collect “is this subject reachable” signals for runtime endpoints, either via active probes or via ecosystem-native “last seen” indicators.

## Applicability

Availability signals are typically applicable to runtime endpoints (agents, services, protocol adapters). They are typically not applicable to catalog-only subjects (for example, a model listing that is not itself a callable endpoint).

## Stored fields (example schema)

Stored in `subject.metadata`:

| Field | Type | Meaning |
| --- | --- | --- |
| `availabilityScore` | number \| null | Ratio in `[0,1]` (preferred) or already-normalized `[0,100]` (implementations MUST document which) |
| `availabilityCheckedAt` | ISO timestamp \| null | Probe timestamp |
| `availabilityLatencyMs` | number \| null | Probe latency (optional) |
| `availabilityReason` | string \| null | Human-readable reason (optional) |
| `availabilitySource` | string \| null | Source label (e.g., `probe`, `metadata`, `marketplace`) |

Optional ecosystem-native recency fields (when active probes are not available):

| Field | Type | Meaning |
| --- | --- | --- |
| `metrics.isOnline` | boolean \| null | Marketplace-provided online flag |
| `metrics.minsFromLastOnline` | number \| null | Minutes since last online |
| `metrics.lastActiveAt` | ISO timestamp \| null | Last active timestamp |

## Notes

- Implementations SHOULD include provenance (source + timestamp) so consumers can understand how “availability” was derived.
- If the ecosystem only provides “last seen”, implementations SHOULD document the exact decay function used when converting last-seen time into a normalized component.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:2ayFqipw6krtLto6minZALXx9qyKUrr17pEPCqtMyHHCZFzMRvSCKhr3dEVcRaM23i;uid=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q;registry=agentverse;proto=agentverse;nativeId=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q`

