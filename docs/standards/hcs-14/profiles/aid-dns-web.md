---
title: HCS-14 Profile - AID Resolution via Web & DNS
description: Profile for resolving uaid:aid identifiers using DNS TXT records and optional cryptographic verification, based on the AID specification.
sidebar_position: 142
---

# HCS-14 Profile: AID Resolution via Web & DNS

**Profile ID:** `hcs-14.profile.aid-dns-web`  
**Status:** Experimental  
**Version:** 0.1.0  
**Applies to:** `uaid:aid:*` where `nativeId` is an FQDN  

## 1. Scope

This profile defines a standardized mechanism for resolving **deterministic AID-based UAIDs** (`uaid:aid:*`) to active agent endpoints using **DNS TXT records and Web-based discovery**.

Specifically, this profile:
- defines how DNS TXT records are used to discover agent endpoints for domain-based identifiers;
- specifies how protocol precedence is determined *within this profile*;
- defines two levels of verification (metadata-based and cryptographic); and
- establishes a clear fallback path to protocol-native discovery when DNS records are absent.

This profile is **optional**. HCS-14 core does not require its implementation and does not prioritize it over other discovery profiles.

## 2. Terminology and Conformance

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119.

A resolver is **conformant** with this profile if it implements all normative requirements in Sections 3–7.

## 3. Applicability (Normative)

This profile applies when all of the following conditions are met:

1. The UAID target is `aid` (i.e., `uaid:aid:*`).
2. The UAID `nativeId` parameter represents a fully qualified domain name (FQDN).
3. The resolver is configured to support this profile.

If any of these conditions are not met, the resolver MUST NOT attempt to apply this profile and SHOULD defer to other supported profiles or protocol-native discovery.

## 4. Discovery via DNS TXT Records (Normative)

### 4.1 DNS record name

Resolvers implementing this profile MUST query the following DNS TXT record:

```

_agent.<nativeId>

```

Example:
```

_agent.example.com

```

### 4.2 TXT record format

Resolvers MUST parse TXT records conforming to **AID v1.x** syntax. At minimum, the following fields are relevant to this profile:

| Tag | Meaning |
|----|--------|
| `v` | AID record version (e.g., `aid1`) |
| `p` or `proto` | Declared agent protocol identifier |
| `u` | Endpoint URI |
| `k` | Multibase-encoded Ed25519 public key (optional) |
| `i` | Key identifier (optional) |

Unknown tags MUST be ignored for forward compatibility.

Example TXT payload (illustrative):

```

v=aid1; p=a2a; u=[https://api.example.com/agent](https://api.example.com/agent); k=z6Mk...; i=key1

````

### 4.3 Protocol precedence (profile-scoped)

If an AID TXT record is successfully retrieved:

- The protocol declared in `p=` or `proto=` MUST take precedence **within this profile** over:
  - the UAID `proto` parameter; and
  - any cached protocol hints.
- This precedence rule applies **only** to resolution performed under this profile and does not override global resolver policy or other profiles.

### 4.4 Endpoint extraction

Resolvers MUST extract the endpoint URI from the `u=` field.

Resolvers MUST validate that:
- the URI scheme is supported by the resolver; and
- the URI is syntactically valid.

Invalid or unsupported URIs MUST result in a profile-level resolution failure.

## 5. Fallback Behavior (Normative)

If the DNS lookup results in **no record** (`NXDOMAIN` or equivalent no-data response):

- the resolver SHOULD proceed with protocol-native discovery mechanisms based on available hints, for example:
  - `/.well-known/agent.json` for A2A;
  - registry-specific APIs;
  - other supported profiles.

If the DNS lookup results in a **record error** (malformed TXT, invalid URI, unsupported protocol):

- the resolver MUST treat this as a resolution failure for this profile and MAY attempt other profiles or discovery mechanisms according to local policy.

## 6. Verification Levels (Normative)

Resolvers implementing this profile SHOULD support both verification levels described below.

### 6.1 Level 1: Metadata Match Verification

**Trigger:**  
- A valid AID TXT record is present; and  
- the record does **not** include a `k=` (public key) field.

**Procedure:**
1. Resolve the endpoint URI from `u=`.
2. Fetch the agent’s canonical metadata using protocol-appropriate means (e.g., `agent.json`, HCS-11 profile).
3. Recompute the AID hash from the canonical metadata.
4. Confirm that the recomputed hash matches the AID component of the UAID.

**Trust model:**  
- DNS integrity and HTTPS/TLS security.
- No cryptographic binding between DNS and the endpoint beyond metadata consistency.

### 6.2 Level 2: Cryptographic Verification (PKA)

**Trigger:**  
- A valid AID TXT record is present; and  
- the record includes a `k=` field.

**Procedure:**
1. Perform the **AID Public Key Authentication (PKA) handshake** as defined in the AID specification.
2. Require the agent server to sign a challenge or response using the private key corresponding to `k=`.
3. Verify the signature using the public key from DNS.
4. Independently perform Level 1 metadata verification.

**Security property:**  
This level cryptographically binds:
- control of the DNS zone; and
- control of the live agent endpoint.

Resolvers MUST fail verification if either the handshake or the metadata match fails.

## 7. Resolver Output Requirements (Normative)

For successful resolution under this profile, resolvers MUST report:

- the resolved endpoint URI;
- the protocol selected via `p=` / `proto=`;
- the verification level achieved (`none`, `metadata`, or `cryptographic`);
- whether protocol precedence was applied from DNS.

Resolvers SHOULD expose this information in resolver metadata.

Illustrative metadata example:

```json
{
  "profile": "hcs-14.profile.aid-dns-web",
  "resolved": true,
  "endpoint": "https://api.example.com/agent",
  "protocol": "a2a",
  "verification": {
    "level": "cryptographic",
    "method": "aid-pka"
  },
  "precedenceSource": "dns"
}
````

## 8. Error Handling (Normative)

If resolution fails under this profile, the resolver MUST return a structured error.

Required error codes:

* `ERR_NOT_APPLICABLE` — profile conditions not met.
* `ERR_NO_DNS_RECORD` — DNS record not found.
* `ERR_INVALID_AID_RECORD` — TXT record malformed or unsupported.
* `ERR_ENDPOINT_INVALID` — endpoint URI invalid or unsupported.
* `ERR_VERIFICATION_FAILED` — metadata or cryptographic verification failed.

Resolvers MAY include additional diagnostic information.

## 9. Security Considerations

* **DNS spoofing:** Implementations SHOULD use DNSSEC where available and SHOULD prefer HTTPS endpoints.
* **Downgrade risk:** Attackers could remove `k=` to force Level 1 verification. Operators SHOULD publish `k=` where cryptographic assurance is required.
* **Endpoint substitution:** Metadata verification mitigates simple redirection attacks but does not replace cryptographic proof.
* **Key rotation:** DNS TTLs and `i=` key identifiers SHOULD be used to manage key rollover safely.

## 10. Examples (Informative)

### 10.1 Example DNS record

```
_agent.example.com TXT
v=aid1; p=a2a; u=https://api.example.com/agent; k=z6MkExampleKey; i=key1
```

### 10.2 Example resolution flow

**Input UAID:**

```
uaid:aid:7bU8...;uid=demo-agent;registry=example;proto=a2a;nativeId=example.com
```

**Execution:**

1. Resolver queries `_agent.example.com`.
2. DNS returns a valid AID record.
3. Resolver selects protocol `a2a` from `p=`.
4. Resolver connects to `https://api.example.com/agent`.
5. Resolver performs PKA handshake and metadata verification.
6. Resolution succeeds with cryptographic assurance.

