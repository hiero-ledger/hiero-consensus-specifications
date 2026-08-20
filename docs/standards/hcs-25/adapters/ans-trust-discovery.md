# HCS-25 (Adapter): ANS Trust Discovery (Informative)

## Purpose

Convert infrastructure-verifiable trust signals from the ANS Trust Index into normalized trust components for ANS-registered agents.

## Contribution

- Adapter id: `ans-trust-discovery`
- Contribution mode: `scoped`
- Suggested weight: `1`
- Default component key: `ans-trust-discovery.certtype`
- Typical applicability: agents registered with `registry=ans` in their UAID
- Typical exclusions: agents in other registries (signals are structurally unavailable)

## Inputs (reference)

Signals are stored under `subject.metadata.ansTrustDiscovery` (see `../signals/ans-trust-discovery.md`).

## Output components (reference)

- `ans-trust-discovery.certtype` in `[0,100]` — certificate type strength
- `ans-trust-discovery.dnssecurity` in `[0,100]` — DNSSEC validation
- `ans-trust-discovery.agentage` in `[0,100]` — registration age
- `ans-trust-discovery.versionstability` in `[0,100]` — version stability
- `ans-trust-discovery.dnsconsistency` in `[0,100]` — DNS consistency
- `ans-trust-discovery.httpsrecord` in `[0,100]` — HTTPS record presence
- `ans-trust-discovery.agentcard` in `[0,100]` — agent-card.json presence
- `ans-trust-discovery.certificatehygiene` in `[0,100]` — certificate health

## Normalization (reference)

All signals are pre-normalized to `[0,100]` by the provider. The adapter passes through each signal score directly as the component value. Null signals are excluded from the adapter total.

Adapter total is the arithmetic mean of non-null components. Minimum 4 of 8 components must be present for the adapter to produce output.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:7bU8xK;uid=b8d9425f-fd9f-47a5-ae5d-8ab51bda04c9;registry=ans;proto=a2a;nativeId=support-agent.example.com;version=v1.0.0`
- UAID mapping: the `uid` parameter (UUID) maps to the ANS `agentId` used for signal collection
