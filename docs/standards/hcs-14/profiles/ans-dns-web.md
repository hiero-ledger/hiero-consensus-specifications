---
title: HCS-14 Profile - ANS Resolution via Web & DNS
description: Profile for resolving uaid:aid identifiers using ANS DNS TXT records, with optional metadata document or direct endpoint resolution, and optional transparency-log verification
sidebar_position: 143
---

# HCS-14 Profile: ANS Resolution via Web & DNS

**Profile ID:** `hcs-14.profile.ans-dns-web`  
**Status:** Draft  
**Version:** 0.1.0  
**Applies to:** `uaid:aid:*` where `registry=ans`, `nativeId` is an FQDN, and `uid` is an ANS agent ID

## Authors

- Connor Snitker ([csnitker@godaddy.com](mailto:csnitker@godaddy.com))  

## 1. Scope

This profile defines a standardized mechanism for resolving **ANS-registered agents** to endpoints and metadata using **DNS TXT records**. Two resolution modes are supported:

- **Direct mode** — the DNS TXT record itself contains a usable endpoint and protocol, enabling lightweight resolution without fetching a separate document.  
- **Fetch mode** — the DNS TXT record references a protocol-native metadata document, which is fetched and parsed to determine endpoints. This is the default mode when `mode` is omitted.

This profile:

- defines DNS discovery for both direct and fetch resolution modes;  
- defines protocol selection rules within this profile; and  
- defines optional verification levels, including optional verification against an HCS-27 transparency log checkpoint stream.

This profile is **optional**. HCS-14 core does not require its implementation and does not prioritize it over other discovery profiles.

## 2. Terminology and Conformance

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119\.

A resolver is **conformant** with this profile if it implements all normative requirements in Sections 3–11.

## 3. Applicability (Normative)

This profile applies when all of the following conditions are met:

1. The UAID target is `aid` (i.e., `uaid:aid:*`).  
2. The UAID `registry` parameter is exactly `ans`.  
3. The UAID `nativeId` parameter represents a fully qualified domain name (FQDN).  
4. The UAID `uid` parameter is present and is a valid ANS agent ID (UUID).  
5. The UAID `version` parameter is present and is a `v`\-prefixed semver 2.0.0 value.  
6. The resolver is configured to support this profile.

If any condition is not met, the resolver MUST NOT attempt to apply this profile and SHOULD defer to other supported profiles or protocol-native discovery.

## 4. Discovery via DNS TXT Records (Normative)

### 4.1 DNS record name

Resolvers implementing this profile MUST query the following DNS TXT record:

```
_ans.<nativeId>
```

Example:

```
_ans.support-agent.example.com
```

### 4.2 `_ans` TXT record format

Resolvers MUST parse the TXT record as a semicolon-delimited key-value list.

The following keys are defined by this profile:

| Key | Required | Meaning |
| :---- | ----: | :---- |
| `v` | yes | Record version. MUST be `ans1` for this profile version. |
| `version` | yes | The ANS registration version of this agent (e.g., `v1.0.0`). MUST be a `v`\-prefixed semver 2.0.0 value. |
| `mode` | no | Resolution mode. MUST be `direct` or `fetch`. Defaults to `fetch` if omitted. |
| `url` | yes | In `fetch` mode: HTTPS URL to the protocol-native metadata document. In `direct` mode: HTTPS URL of the agent endpoint directly. |
| `p` | conditional | The agent communication protocol (e.g., `mcp`, `a2a`, `http`). Required when `mode=direct`. |

Unknown keys MUST be ignored for forward compatibility.

If `v` is missing or not equal to `ans1`, the resolver MUST treat this profile as not applicable for the given identifier.

If `version` is missing, the resolver MUST fail resolution with `ERR_INVALID_ANS_RECORD`.

### 4.3 Version matching

Resolvers MUST compare the UAID `version` parameter against the `version` value in the TXT record. Both values MUST conform to Semantic Versioning 2.0.0 (semver.org) and MUST be prefixed with `v` (e.g., `v1.0.0`). Comparison MUST follow semver 2.0.0 precedence rules.

If the versions do not match, the resolver MUST fail resolution with `ERR_VERSION_MISMATCH`.

If either value cannot be parsed as a valid semver 2.0.0 version, the resolver MUST fail resolution with `ERR_INVALID_ANS_RECORD`.

### 4.4 Mode determination

Resolvers MUST determine the resolution mode as follows:

1. If `mode=direct` is present, apply **Direct Mode** (Section 5).  
2. If `mode=fetch` is present, or if `mode` is absent, apply **Fetch Mode** (Section 6).  
3. If `mode` contains any other value, the resolver MUST fail resolution with `ERR_INVALID_ANS_RECORD`.

If `mode=direct` and `p` is absent, the resolver MUST fail resolution with `ERR_INVALID_ANS_RECORD`.

## 5. Direct Mode Resolution (Normative)

When operating in direct mode, resolvers MUST resolve the agent endpoint directly from the DNS TXT record without fetching any additional document.

### 5.1 Required fields

Resolvers MUST verify that `url` and `p` are both present in the TXT record. If either is missing or malformed, the resolver MUST fail resolution with `ERR_INVALID_ANS_RECORD`.

The `url` value MUST be an `https:` URL. If it is not, the resolver MUST fail resolution with `ERR_INVALID_ANS_RECORD`.

### 5.2 Stable FQDN anchoring

Resolvers MUST validate that the host of the `url` value is exactly equal to the UAID `nativeId` value. If it does not match, the resolver MUST fail resolution with `ERR_ENDPOINT_NOT_ANCHORED`.

### 5.3 Protocol selection

Resolvers MUST use the `p` value from the TXT record as the resolved protocol. If the UAID includes a `proto` parameter, resolvers SHOULD verify that it matches the `p` value in the TXT record; a mismatch SHOULD be surfaced as a warning in resolver metadata but MUST NOT cause resolution to fail.

### 5.4 Output

On successful direct mode resolution, resolvers MUST report the resolved endpoint URL, the `p` value, and that resolution was performed via direct mode (see Section 9).

## 6. Fetch Mode Resolution (Normative)

When operating in fetch mode, resolvers MUST fetch and parse a protocol-native metadata document to determine the agent endpoint.

### 6.1 Required fields

Resolvers MUST verify that `url` is present in the TXT record and SHOULD be an `https:` URL. If it is missing or invalid, the resolver MUST fail resolution with `ERR_INVALID_ANS_RECORD`.

### 6.2 Protocol determination

If `p` is present in the TXT record, resolvers MUST use that value as the requested protocol. Otherwise, resolvers MUST derive the protocol from the fetched metadata document using protocol-native rules. If no protocol can be determined, the resolver MUST fail resolution with `ERR_ENDPOINT_NOT_FOUND`.

### 6.3 Metadata document retrieval

Resolvers MUST fetch the document at `url`. The document format and schema are defined by the protocol identified in Section 6.2 — this profile does not prescribe a document structure. Examples of protocol-native metadata documents include A2A Agent Cards and MCP server manifests.

If the document cannot be fetched or cannot be parsed according to the protocol-native format, the resolver MUST fail resolution with `ERR_METADATA_INVALID`.

### 6.4 Endpoint extraction

Resolvers MUST extract an endpoint URL from the metadata document using rules defined by the protocol identified in Section 6.2. This profile does not define how endpoint extraction is performed for any specific protocol.

If no usable endpoint can be extracted, the resolver MUST fail resolution with `ERR_ENDPOINT_NOT_FOUND`.

### 6.5 Stable FQDN anchoring

For each extracted endpoint URL, resolvers MUST validate that the URL host is exactly equal to the UAID `nativeId` value. If an endpoint URL host does not equal `nativeId`, the resolver MUST reject that endpoint candidate.

If all endpoint candidates are rejected by this rule, the resolver MUST fail resolution with `ERR_ENDPOINT_NOT_ANCHORED`.

## 7. Transparency Log Discovery and Verification (Normative)

ANS transparency verification is discovered via a second DNS TXT record, `_ans-badge.<nativeId>`, which is independent of the `_ans` resolution record. This separation means transparency verification is available regardless of resolution mode.

### 7.1 Badge DNS record

Resolvers MAY query the following DNS TXT record to discover the transparency badge endpoint for an agent:

```
_ans-badge.<nativeId>
```

Example:

```
_ans-badge.support-agent.example.com
```

The badge TXT record follows the same semicolon-delimited key-value format as the `_ans` record. The following keys are defined:

| Key | Required | Meaning |
| :---- | ----: | :---- |
| `v` | yes | Record version. MUST be `ans-badge1` for this profile version. |
| `version` | yes | Agent version this badge record corresponds to. MUST be a `v`\-prefixed semver 2.0.0 value (e.g., `v1.0.0`). |
| `url` | yes | HTTPS URL to the badge endpoint for this agent. |

If `v` is missing or not equal to `ans-badge1`, the resolver MUST ignore this record and treat transparency verification as unavailable.

If `version` does not match the `version` value in the `_ans` record, the resolver MUST treat transparency verification as unavailable and MUST NOT proceed with badge verification.

### 7.2 Badge endpoint response

The badge endpoint at `url` returns a JSON document containing the agent's transparency log entry and Merkle inclusion proof. The response includes at minimum:

- `status` — the current registration status of the agent (e.g., `ACTIVE`, `REVOKED`).  
- `payload` — the log entry payload, which MUST contain `ansName` and `version` fields binding the entry to a specific agent identity.  
- `merkleProof` — the inclusion proof object as defined by HCS-27, containing `leafHash`, `leafIndex`, `treeSize`, `rootHash`, `rootSignature`, and `path`.  
- `hcs27TopicId` — the Hedera topic ID where HCS-27 checkpoint messages for this agent's transparency log are published (e.g., `0.0.12345`). Used for Level 2c verification.

Resolvers MUST treat a badge endpoint fetch failure as transparency verification unavailable, and MUST NOT fail endpoint resolution as a result.

### 7.3 Verification levels

Transparency verification is optional and resolvers SHOULD expose which level was achieved in resolver output (see Section 9).

**Level 2a — Status and binding check**

The resolver MUST:

1. Verify that `status` in the badge response is not a revoked or otherwise terminal state.  
2. Verify that `payload.ansId` matches the UAID `uid` parameter.  
3. Verify that `payload.version` matches the UAID `version` parameter.

If any check fails, the resolver MUST fail transparency verification with `ERR_TRANSPARENCY_VERIFICATION_FAILED`.

**Level 2b — Merkle inclusion proof verification**

The resolver MUST perform Level 2a checks first, then additionally verify the Merkle inclusion proof contained in `merkleProof` using the algorithms defined in HCS-27 Merkle Tree Profile and Proofs. Specifically:

1. Verify the `rootSignature` is a valid signed tree head (STH) for the given `rootHash` and `treeSize`.  
2. Verify the inclusion proof (`leafIndex`, `path`, `treeSize`, `rootHash`) using the HCS-27 `VerifyInclusion` algorithm.

**Level 2c — HCS checkpoint verification (Optional)**

For the highest assurance, resolvers MAY additionally verify that `rootHash` and `treeSize` from the badge response correspond to a published HCS-27 checkpoint message on the Hedera network. The checkpoint topic is identified by the `hcs27TopicId` field in the badge response. Resolvers MUST treat a missing or invalid `hcs27TopicId` as Level 2c unavailable; this MUST NOT affect Level 2a or 2b verification. The mechanism for reading HCS checkpoint messages is out of scope for this profile and is defined by HCS-27.

## 8. Verification Levels (Normative)

### 8.1 Level 1: DNS and Endpoint Verification

For Level 1 verification, resolvers MUST:

1. Enforce stable FQDN anchoring (Section 5.2 or Section 6.5, as applicable).  
2. In fetch mode, confirm the metadata document was successfully retrieved and parsed.

Resolvers SHOULD use DNSSEC validation for TXT retrieval where available and SHOULD prefer HTTPS endpoints with strong TLS configurations.

### 8.2 Level 2: Transparency Verification (Optional)

Transparency verification is performed against the `_ans-badge` record as defined in Section 7\. Three sub-levels are available in increasing order of assurance:

- **Level 2a** — Status and binding check (Section 7.3).  
- **Level 2b** — Merkle inclusion proof verification (Section 7.3).  
- **Level 2c** — HCS checkpoint verification (Section 7.3).

Resolvers MUST NOT require transparency verification for baseline endpoint resolution. Achieving Level 1 is sufficient for resolution to succeed.

## 9. Resolver Output Requirements (Normative)

For successful resolution under this profile, resolvers MUST report:

- the resolved endpoint URL(s);  
- the requested protocol value used for selection;  
- the resolution mode used (`direct` or `fetch`);  
- in fetch mode, the metadata document URL (`url`) used;  
- whether Level 1 verification was achieved; and  
- whether transparency verification was attempted and, if so, which level was achieved (2a, 2b, or 2c).

## 10. Error Handling (Normative)

If resolution fails under this profile, the resolver MUST return a structured error.

Required error codes:

* `ERR_NOT_APPLICABLE` — profile conditions not met.  
* `ERR_NO_DNS_RECORD` — DNS record not found.  
* `ERR_INVALID_ANS_RECORD` — TXT record malformed, unsupported version, missing required fields, or invalid mode.  
* `ERR_VERSION_MISMATCH` — the UAID `version` parameter does not match the `version` value in the `_ans` DNS TXT record (after validating both as `v`\-prefixed SemVer 2.0.0).
* `ERR_METADATA_INVALID` — metadata document could not be fetched or parsed (fetch mode only).  
* `ERR_ENDPOINT_NOT_FOUND` — no usable endpoint for requested protocol.  
* `ERR_ENDPOINT_NOT_ANCHORED` — endpoint host does not match `nativeId`.  
* `ERR_TRANSPARENCY_VERIFICATION_FAILED` — badge status check failed, binding mismatch, or Merkle proof verification failed.

Resolvers MAY include additional diagnostic information.

## 11. Security Considerations

* **DNS spoofing:** Implementations SHOULD validate DNSSEC where available. Direct mode is particularly sensitive to DNS spoofing since no metadata document provides a secondary binding check.  
* **Endpoint substitution:** Stable FQDN anchoring reduces redirection risk but does not replace cryptographic proof.  
* **Direct mode trust:** Direct mode provides a lower verification assurance than fetch mode because the endpoint is asserted solely via DNS. Relying parties with strict trust requirements SHOULD prefer fetch mode or require Level 2 transparency verification.  
* **Centralization:** This profile provides discovery and optional verification hooks; it does not itself define a decentralization or governance model for registries.  
* **Downgrade risk:** Implementations SHOULD expose whether transparency verification was performed so relying parties can enforce policy.

## 12. Examples (Informative)

### 12.1 Direct mode DNS record

```
_ans.support-agent.example.com TXT
v=ans1; version=v1.0.0; mode=direct; url=https://support-agent.example.com/mcp; p=mcp
```

### 12.2 Fetch mode DNS record

```
_ans.support-agent.example.com TXT
v=ans1; version=v1.0.0; mode=fetch; url=https://support-agent.example.com/agent-card.json
```

Or equivalently, omitting `mode` (defaults to fetch):

```
_ans.support-agent.example.com TXT
v=ans1; version=v1.0.0; url=https://support-agent.example.com/agent-card.json
```

### 12.3 Badge DNS record

```
_ans-badge.support-agent.example.com TXT
v=ans-badge1; version=v1.0.0; url=https://transparency.ans.example.com/v1/agents/b8d9425f-fd9f-47a5-ae5d-8ab51bda04c9
```

### 12.4 Example UAID

```
uaid:aid:7bU8...;uid=b8d9425f-fd9f-47a5-ae5d-8ab51bda04c9;registry=ans;version=v1.0.0;proto=a2a;nativeId=support-agent.example.com
```
