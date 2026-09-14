# HCS-25 (Signal): ANS Trust Discovery (Informative)

## Purpose

Collect infrastructure trust signals from the Agent Name Service (ANS) Trust Index, providing machine-verifiable identity and integrity measurements for registered AI agents.

## Applicability

- Applied to agents registered with `registry=ans` in their UAID (see HCS-14 `ans-dns-web` profile).
- Excludes agents in other registries (signals are structurally unavailable from the ANS Trust Index).

## Collection method

Signal adapters query an ANS Trust Index provider's agent detail endpoint.

### Reference endpoint format

```
GET {ans_base_url}/v1/ans/registered-agents/{agent_id}
```

Where `{agent_id}` is the agent identifier known to the ANS provider.

### Example response (signals portion)

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
      "score": 80,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "identity",
      "evidence": {
        "dnssecValid": true,
        "tlsaPresent": true,
        "raBadgePresent": false
      }
    },
    "agentage": {
      "score": 60,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "registeredAt": "2026-03-01T00:00:00Z",
        "ageDays": 170
      }
    },
    "versionstability": {
      "score": 100,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "versionChanges": 1,
        "observationWindowDays": 90
      }
    },
    "dnsconsistency": {
      "score": 70,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "ansMatchVerified": true,
        "raBadgeMatchVerified": false,
        "domainResolved": true
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
      "score": 95,
      "missing": false,
      "computedAt": "2026-08-18T01:58:17.623Z",
      "pillar": "integrity",
      "evidence": {
        "daysToExpiry": 245,
        "serverCertStatus": "matched"
      }
    }
  }
}
```

Each signal returns a `score` in `[0,100]`, a `missing` flag, a `computedAt` timestamp, a `pillar` categorization, and an `evidence` object with signal-specific observation data. When `missing` is `true`, the signal could not be computed and `score` should be treated as null. The `evidence` field contents are illustrative; actual keys vary by signal and may evolve.

### Providers

| Provider | Base URL | Scope |
| --- | --- | --- |
| GoDaddy ANS | `https://api.godaddy.com` | ANS-registered agents |

_Additional providers will be listed as they implement the signal schema._

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
| `ans-trust-discovery.certificatehygiene` | number \| null | Certificate health (expiry, completeness, consistency, rotation) |
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
