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
| `ans-trust-discovery.certtype` | number \| null | Certificate type strength (DV=40, OV=70, EV=100) |
| `ans-trust-discovery.dnssecurity` | number \| null | DNSSEC validation (0 or 100) |
| `ans-trust-discovery.agentage` | number \| null | Registration age (logarithmic, 0-100) |
| `ans-trust-discovery.versionstability` | number \| null | Version churn rate (inverse, 0-100) |
| `ans-trust-discovery.dnsconsistency` | number \| null | Cross-resolver DNS agreement (0-100) |
| `ans-trust-discovery.httpsrecord` | number \| null | HTTPS DNS record presence (0 or 100) |
| `ans-trust-discovery.agentcard` | number \| null | agent-card.json reachable (0 or 100) |
| `ans-trust-discovery.certificatehygiene` | number \| null | Certificate health (starts at 100, deducts for issues) |
| `ansTrustDiscoveryStatus` | `ok` \| `missing` \| `error` \| `stale` | Overall collection status |
| `ansTrustDiscoveryUpdatedAt` | ISO timestamp | Refresh time |

Per-signal status is conveyed via null (missing) vs numeric (ok). Signals measure publicly observable infrastructure state (DNS, TLS, HTTPS records) verified from multiple resolvers.

## Notes

- Signals refresh every 24 hours; infrastructure state is slowly-changing.
- Certificate type is anchored to CA-issued certificates (DV/OV/EV) — cannot be self-asserted.
- Agent age is immutable once established.
- Registration records are anchored in an HCS-27 Merkle-tree transparency log, providing tamper evidence.
- The reference provider verifies DNS from multiple geographically distributed resolvers.

## Production example (Registry Broker; informative)

- Endpoint: `https://hol.org/registry/api/v1/agents/{uaid}`
- Example UAID: `uaid:aid:7bU8xK;uid=b8d9425f-fd9f-47a5-ae5d-8ab51bda04c9;registry=ans;proto=a2a;nativeId=support-agent.example.com;version=v1.0.0`
