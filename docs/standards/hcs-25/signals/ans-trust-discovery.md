# HCS-25 (Signal): ANS Trust Discovery (Informative)

## Purpose

Collect per-agent infrastructure trust signals from the Agent Name Service (ANS) Trust Index, providing machine-verifiable identity and integrity measurements for registered AI agents.

## Applicability

Applied to agents registered with `registry=ans` in their UAID (see HCS-14 `ans-dns-web` profile). Signals are structurally unavailable for agents in other registries.

## Background

Unlike retrospective trust signals (benchmarks, historical ratings) or self-reported metrics (marketplace scores), ANS Trust Discovery signals measure **observable infrastructure state** from the public internet: DNS configuration, TLS certificate properties, HTTPS record presence, and registration provenance.

These signals answer "is this agent's infrastructure configured correctly and consistently?" rather than "how has this agent performed historically?"

Key properties:

- **Infrastructure-verifiable**: All signals derive from publicly observable DNS, TLS, and HTTP state
- **CA-anchored identity**: Certificate type signals (DV/OV/EV) require issuance from trusted Certificate Authorities
- **Transparency-log-backed**: Agent registration records are anchored in an HCS-27 Merkle tree, providing tamper evidence for registration state changes
- **Slowly-changing**: Infrastructure signals measure configuration compliance, not transient behavior

## Collection method

Signal adapters query the ANS Trust Index provider's agent detail endpoint. The endpoint returns pre-normalized scores (0-100) for each signal, along with freshness timestamps, missing-data flags, and supporting evidence.

### Reference endpoint format

```
GET {ans_base_url}/v1/ans/registered-agents/{agentId}
```

Where `{agentId}` is the `uid` parameter from the agent's UAID (a UUID assigned at registration time).

Example request:

```
GET https://search.agent-name-service.godaddy.com/v1/ans/registered-agents/b8d9425f-fd9f-47a5-ae5d-8ab51bda04c9
```

Example response (signals portion):

```json
{
  "signals": {
    "certtype": {
      "score": 100,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "identity",
      "evidence": {
        "certType": "EV",
        "issuer": "DigiCert EV RSA CA G2"
      }
    },
    "dnssecurity": {
      "score": 100,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "identity",
      "evidence": {
        "dnssecEnabled": true,
        "validationChain": ["DS", "DNSKEY", "RRSIG"]
      }
    },
    "agentage": {
      "score": 75,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "registeredAt": "2026-03-01T00:00:00Z",
        "ageDays": 170
      }
    },
    "versionstability": {
      "score": 90,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "versionChanges": 2,
        "observationWindowDays": 90
      }
    },
    "dnsconsistency": {
      "score": 100,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "resolversQueried": 4,
        "resolversAgreeing": 4
      }
    },
    "httpsrecord": {
      "score": 100,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "httpsRecordPresent": true
      }
    },
    "agentcard": {
      "score": 100,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "metadata",
      "evidence": {
        "cardPresent": true,
        "source": "well-known"
      }
    },
    "certificatehygiene": {
      "score": 100,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "daysToServerExpiry": 245,
        "daysToIdentityExpiry": 245,
        "serverCertificateStatus": "matched"
      }
    }
  }
}
```

### Minimum provider requirements

A conforming ANS Trust Index provider MUST:

1. Verify infrastructure signals from the public internet (DNS queries, TLS handshakes, HTTP requests) — not self-reported by the agent
2. Return structured scores with calibrated 0-100 ranges per signal
3. Provide per-signal provenance via `computedAt` timestamp and `evidence` object
4. Anchor agent registration records in an HCS-27 transparency log
5. Return `missing: true` for any signal that could not be computed (e.g., DNS timeout, unreachable host)

A conforming ANS Trust Index provider SHOULD:

1. Refresh signals on a cadence no longer than 24 hours
2. Provide Merkle inclusion proof verification for registration state via HCS-27
3. Verify DNS from multiple geographically distributed resolvers
4. Validate TLS certificate chains against the Mozilla Root CA program or equivalent

## Stored fields (example schema)

Stored in `subject.metadata.ansTrustDiscovery`:

| Field | Type | Meaning |
| --- | --- | --- |
| `certtype` | object | Certificate type strength signal |
| `dnssecurity` | object | DNSSEC validation signal |
| `agentage` | object | Registration age signal |
| `versionstability` | object | Version churn rate signal |
| `dnsconsistency` | object | Cross-resolver DNS consistency signal |
| `httpsrecord` | object | HTTPS DNS record presence signal |
| `agentcard` | object | agent-card.json metadata presence signal |
| `certificatehygiene` | object | Certificate health and expiry signal |
| `updatedAt` | ISO timestamp | When the signal set was last refreshed |

### Per-signal object

Each signal object has a uniform structure:

| Field | Type | Meaning |
| --- | --- | --- |
| `score` | number | Normalized score in `[0,100]` |
| `missing` | boolean | Whether the signal could not be computed |
| `computedAt` | ISO timestamp | When this signal was last evaluated |
| `pillar` | string | Signal category: `identity`, `integrity`, or `metadata` |
| `evidence` | object | Provider-specific evidence payload (contents vary per signal) |

### Signal scoring semantics

| Signal | Pillar | Range | Scoring method |
| --- | --- | --- | --- |
| `certtype` | identity | 0-100 | DV=40, OV=70, EV=100; 0 if no certificate |
| `dnssecurity` | identity | 0 or 100 | Binary: DNSSEC chain validated=100, absent=0 |
| `agentage` | integrity | 0-100 | Logarithmic growth from registration timestamp |
| `versionstability` | integrity | 0-100 | Inverse of version churn over observation window |
| `dnsconsistency` | integrity | 0-100 | Percentage agreement across queried resolvers |
| `httpsrecord` | integrity | 0 or 100 | Binary: HTTPS DNS record present=100, absent=0 |
| `agentcard` | metadata | 0 or 100 | Binary: agent-card.json reachable=100, absent=0 |
| `certificatehygiene` | integrity | 0-100 | Starts at 100; deducts for expired, mismatched, or weak certificates |

## Signal provenance

Each ANS Trust Index provider is identified by:

- **Provider identifier**: The ANS registry operator (e.g., `godaddy-ans`)
- **Method**: Infrastructure verification via public internet probes and CA chain validation
- **Attestation**: HCS-27 transparency log (Merkle-tree anchored registration records)

## Refresh cadence (informative)

Recommended: every 24 hours.

ANS signals measure slowly-changing infrastructure state (DNS records, TLS certificates, version stability). Daily freshness is adequate and matches the provider's internal refresh cycle. More frequent polling yields no additional signal value.

## Gaming resistance considerations

ANS Trust Discovery signals are inherently resistant to gaming because they measure infrastructure state that requires real-world changes to influence:

- **Certificate type**: Upgrading from DV to OV or EV requires identity verification by a trusted CA — cannot be self-asserted
- **DNSSEC**: Requires actual DNSSEC deployment across the DNS delegation chain
- **Agent age**: Immutable once established; cannot be accelerated
- **Version stability**: Gaming requires refraining from version changes (which is the desired behavior)
- **DNS consistency**: Requires consistent configuration across authoritative nameservers
- **Certificate hygiene**: Requires maintaining valid, non-expired, correctly-matched certificates
- **HCS-27 anchoring**: Registration state changes are logged in an append-only Merkle tree; falsifying history requires breaking the hash chain

Binary signals (dnssecurity, httpsrecord, agentcard) can be trivially satisfied by enabling the feature. This is intentional — the signal measures compliance with best practice, not difficulty of achievement.

## Limitations

- Scoped to ANS-registered agents only; signals are structurally unavailable for other registries
- Binary signals provide coarse granularity (only distinguish compliant/non-compliant)
- Agent age rewards longevity but does not directly measure quality or capability
- Certificate hygiene uses a deduction model; a single expired certificate can dominate the score
- Single-provider dependency: all signals originate from one ANS Trust Index operator

## License

Apache-2.0
