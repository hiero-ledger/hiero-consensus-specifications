# HCS-25 (Adapter): ANS Trust Discovery (Informative)

## Purpose

Convert infrastructure-verifiable trust signals from the ANS Trust Index into normalized trust components, providing an identity and integrity dimension to the composite AI Trust Score for ANS-registered agents.

While existing adapters measure **historical performance** (benchmarks, ratings) or **adoption** (downloads, stars, marketplace metrics), this adapter measures **ongoing infrastructure health** — whether the agent's DNS, TLS, and identity configuration meets operational best practices and remains consistent over time.

## Contribution

- Adapter id: `ans-trust-discovery`
- Contribution mode: `scoped`
- Suggested weight: `1`
- Typical applicability: agents registered with `registry=ans` in their UAID
- Typical exclusions: agents in other registries (signals are structurally unavailable from the ANS Trust Index)

## Inputs (reference)

- `subject.metadata.ansTrustDiscovery` (see `../signals/ans-trust-discovery.md`)

## Output components

- `ans-trust-discovery.certtype` in `[0,100]` — certificate type strength
- `ans-trust-discovery.dnssecurity` in `[0,100]` — DNSSEC validation
- `ans-trust-discovery.agentage` in `[0,100]` — registration age
- `ans-trust-discovery.versionstability` in `[0,100]` — version stability
- `ans-trust-discovery.dnsconsistency` in `[0,100]` — DNS consistency
- `ans-trust-discovery.httpsrecord` in `[0,100]` — HTTPS record presence
- `ans-trust-discovery.agentcard` in `[0,100]` — agent-card.json presence
- `ans-trust-discovery.certificatehygiene` in `[0,100]` — certificate health

## Normalization (reference)

All ANS Trust Index signals are pre-normalized to `[0,100]` by the provider. The adapter passes through each signal score directly as the component value:

```
component_value = signal.score   // already in [0,100], no transformation needed
```

If a signal has `missing: true`, that component is excluded from the adapter total (treated as missing data per HCS-25 §Missing and Stale Signals).

### Per-component scoring semantics (informative)

These are performed by the ANS provider, not by the adapter. Documented here for transparency:

| Component | Method |
| --- | --- |
| `certtype` | DV=40, OV=70, EV=100 (certificate type strength from CA-issued certificates) |
| `dnssecurity` | Binary: DNSSEC chain validated=100, absent=0 |
| `agentage` | Logarithmic growth from registration timestamp |
| `versionstability` | Inverse of version churn rate over 90-day observation window |
| `dnsconsistency` | Percentage agreement across multiple public DNS resolvers |
| `httpsrecord` | Binary: HTTPS DNS record present=100, absent=0 |
| `agentcard` | Binary: agent-card.json reachable at well-known path=100, absent=0 |
| `certificatehygiene` | Starts at 100; deducts for expired (-20 to -25 per cert), mismatched (-25), or untrusted (-30) certificates |

## Adapter total

Arithmetic mean of all non-missing component scores:

```
total = round( sum(component_i for non-missing components) / count(non_missing_components) )
```

Minimum threshold: at least 4 of 8 components must be non-missing for the adapter to produce output. Below this threshold, signal coverage is insufficient and the adapter returns no score.

## Applicability guidance

- **Include**: Any agent with `registry=ans` in its UAID
- **Exclude**: Agents in other registries (ANS signals are structurally unavailable)
- **Exclude**: Agents with fewer than 4 non-missing signals (insufficient coverage)
- **UAID mapping**: The `uid` parameter of the UAID maps to the `{agentId}` path parameter in the ANS API. The `uid` is a UUID assigned at registration and is immutable across agent version changes.

## Relationship to other adapters

| Adapter | What it measures | Signal source | Temporal scope |
| --- | --- | --- | --- |
| `availability` | Is the agent reachable? | Active probe | Current |
| `ethos` | Credibility contribution | Third-party platform | Retrospective |
| `output-verification` | Is THIS output correct? | Verification provider | Per-decision |
| `oss-popularity` | Community adoption | GitHub/npm | Retrospective |
| **`ans-trust-discovery`** | **Infrastructure identity & integrity** | **ANS Trust Index** | **Current (24h refresh)** |

Key differentiator: `ans-trust-discovery` is the only adapter that measures **identity strength** (certificate type, DNSSEC) and **infrastructure integrity** (DNS consistency, certificate health, version stability) from a DNS-and-PKI-based agent registry.

## Security considerations

- **Single-provider dependency**: All 8 signals originate from one ANS Trust Index operator. Multi-provider cross-referencing is not applicable (unlike `output-verification`). The HCS-27 transparency log provides independent verifiability of registration state.
- **Scoped contribution mode**: The adapter only participates in scoring for `registry=ans` agents, preventing meaningless scores for agents where signals are unavailable.
- **Binary signal gaming**: Binary signals (dnssecurity, httpsrecord, agentcard) can be trivially satisfied by enabling the feature. This is intentional — the signal measures compliance with operational best practice, not difficulty of achievement.
- **Certificate type anchoring**: Certificate type scores are anchored to CA-issued certificates (DV/OV/EV). Self-signed certificates do not contribute a certtype score, preventing self-assertion of identity strength.
- **Transparency log verification**: Implementations SHOULD verify HCS-27 Merkle inclusion proofs for registration records when available. This provides tamper evidence independent of the ANS provider.

## Multi-provider support

Not applicable. ANS Trust Discovery signals are structurally tied to a single ANS registry operator — the registry that issued the agent's certificate and manages its DNS records. Independent verification is provided through HCS-27 transparency log proofs rather than through multi-provider aggregation.

## Reference implementations (informative)

| Provider | Method | Attestation |
| --- | --- | --- |
| GoDaddy ANS Trust Index | Infrastructure verification via public internet probes (DNS, TLS, HTTP) + CA chain validation | HCS-27 transparency log (Merkle-tree anchored registration records) |

_Additional conforming providers will be listed as they implement the signal schema._

## Production example (Registry Broker; informative)

- Signal endpoint: `GET {ans_base_url}/v1/ans/registered-agents/{uid}`
- Example UAID: `uaid:aid:7bU8xK;uid=b8d9425f-fd9f-47a5-ae5d-8ab51bda04c9;registry=ans;proto=a2a;nativeId=support-agent.example.com;version=v1.0.0`

## License

Apache-2.0
