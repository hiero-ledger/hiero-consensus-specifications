# HCS-25 (Signal): ANS Trust Discovery (Informative)

## Purpose

Collect infrastructure trust signals from the Agent Name Service (ANS) Trust Index, providing machine-verifiable identity and integrity measurements for registered AI agents.

## Applicability

- Applied to agents registered with `registry=ans` in their UAID (see HCS-14 `ans-dns-web` profile).
- Excludes agents in other registries (signals are structurally unavailable from the ANS Trust Index).

## Stored fields (example schema)

Stored in `subject.metadata.ansTrustDiscovery`:

| Field | Type | Meaning |
| --- | --- | --- |
| `ans-trust-discovery.certtype` | number \| null | Certificate type strength (higher for OV/EV vs DV) |
| `ans-trust-discovery.dnssecurity` | number \| null | DNS security posture (DNSSEC, TLSA, related records) |
| `ans-trust-discovery.agentage` | number \| null | Time since initial registration |
| `ans-trust-discovery.versionstability` | number \| null | Version change frequency (less churn = higher score) |
| `ans-trust-discovery.dnsconsistency` | number \| null | DNS record consistency and attestation match |
| `ans-trust-discovery.httpsrecord` | number \| null | HTTPS DNS record presence (0 or 100) |
| `ans-trust-discovery.agentcard` | number \| null | agent-card.json reachable (0 or 100) |
| `ans-trust-discovery.certificatehygiene` | number \| null | Certificate health (expiry, consistency, rotation) |
| `ansTrustDiscoveryStatus` | `ok` \| `missing` \| `error` \| `stale` | Overall collection status |
| `ansTrustDiscoveryUpdatedAt` | ISO timestamp | Refresh time |

Per-signal status is conveyed via null (missing) vs numeric (ok). All numeric values are in `[0,100]`. Signals measure publicly observable infrastructure state (DNS, TLS, HTTPS records).

## Notes

- Signals refresh every 24 hours; infrastructure state is slowly-changing.
- Certificate type scores are anchored to CA-issued certificates — cannot be self-asserted.
- Registration timestamps cannot be backdated; agent age can only grow organically.
- Registration records are anchored in an HCS-27 Merkle-tree transparency log, providing tamper evidence.
- Scoring methodology is configurable by the provider; specific formulas may evolve independently of this signal schema.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:7bU8xK;uid=b8d9425f-fd9f-47a5-ae5d-8ab51bda04c9;registry=ans;proto=a2a;nativeId=support-agent.example.com;version=v1.0.0`
