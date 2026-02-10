# HCS-25 (Adapter): Availability (Informative)

## Purpose

Score whether a subject is reachable/recently active. Intended for runtime endpoints.

## Contribution

- Adapter id: `availability`
- Contribution mode: `universal`
- Suggested weight: `1`
- Typical applicability: runtime endpoints where a “reachable” notion is meaningful
- Typical exclusions: model catalogs where reachability is not defined (e.g., `openrouter`, `near-ai`)

## Inputs (reference)

Uses `subject.metadata`:

- `availabilityScore` (ratio `[0,1]`, when present)
- `availabilitySource` (string, used to decide whether to trust `availabilityScore`)
- For some ecosystems, uses `metadata.metrics` fields such as:
  - `isOnline` (boolean)
  - `minsFromLastOnline` (number)
  - `lastActiveAt` (timestamp string)

## Output components

- `availability.uptime` in `[0,100]`

## Related signals

See `../signals/availability.md`.

## Normalization (reference)

- Prefer direct `availabilityScore` when present.
- If direct probes are not available, a deterministic recency decay may be used (e.g., linear decay over 24 hours based on last-seen).

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:2ayFqipw6krtLto6minZALXx9qyKUrr17pEPCqtMyHHCZFzMRvSCKhr3dEVcRaM23i;uid=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q;registry=agentverse;proto=agentverse;nativeId=agent1qghw9n306v4jynrxtj07an03g8xtzmjcpmpl0l4cggmvqmjmrqr6c0np39q`
