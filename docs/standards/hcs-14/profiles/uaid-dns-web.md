---
title: HCS-14 Profile - UAID Resolution via `_uaid` DNS TXT
description: Profile for binding DNS names to HCS-14 identifiers via `_uaid.<nativeId>` TXT records and dispatching to downstream resolution profiles.
sidebar_position: 141
---

# HCS-14 Profile: UAID Resolution via `_uaid` DNS TXT

**Profile ID:** `hcs-14.profile.uaid-dns-web`
**Status:** Experimental
**Version:** 0.1.0
**Applies to:** `uaid:aid:*` and `uaid:did:*` where `nativeId` is an FQDN

## 1. Scope

This profile defines DNS-based UAID identity binding using a dedicated TXT record at:

```
_uaid.<nativeId>
```

The TXT payload carries HCS-14 identifier components and routing parameters directly, so resolvers can validate that a domain explicitly advertises a specific UAID and then hand off to a downstream profile.

This profile:
- defines `_uaid` TXT record structure and validation;
- defines deterministic reconstruction of UAID from TXT fields;
- defines dispatch to downstream resolver profiles.

This profile is optional. HCS-14 core does not require support for it.

## 2. Terminology and Conformance

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119.

A resolver is conformant with this profile if it implements all normative requirements in Sections 3–10.

## 3. Applicability (Normative)

This profile applies when all conditions are met:

1. Input is a valid `uaid:aid:*` or `uaid:did:*`.
2. UAID `nativeId` is present and is a valid FQDN.
3. Resolver is configured to support this profile.

If any condition fails, resolver MUST NOT apply this profile.

## 4. Discovery via `_uaid` TXT (Normative)

### 4.1 DNS record name

Resolvers implementing this profile MUST query:

```
_uaid.<nativeId>
```

Example:

```
_uaid.support-agent.example.com
```

### 4.2 TXT record format

Resolvers MUST parse TXT payload as semicolon-delimited key-value fields.

Required keys:

| Key | Required | Meaning |
|---|---:|---|
| `target` | yes | HCS-14 target; MUST be `aid` or `did`. |
| `id` | yes | HCS-14 identifier component (the part after `uaid:{target}:`). |
| `uid` | yes | HCS-14 `uid` parameter value. |
| `registry` | yes | HCS-14 `registry` parameter value. |
| `proto` | yes | HCS-14 `proto` parameter value. |
| `nativeId` | yes | HCS-14 `nativeId` parameter value. |

Optional keys:

| Key | Meaning |
|---|---|
| `domain` | HCS-14 `domain` parameter value, if used by the UAID. |
| `src` | HCS-14 `src` parameter value for `uaid:did:*` sanitized IDs. |

Unknown keys MUST be ignored for forward compatibility.

This profile defines no profile-local TXT version field. Compatibility is governed by profile ID and specification version.

### 4.3 Field validation

Resolvers MUST validate:

1. `target` is exactly `aid` or `did`.
2. `id` is non-empty.
3. `uid`, `registry`, `proto`, `nativeId` are non-empty.
4. `nativeId` equals the queried `<nativeId>` (case-insensitive DNS comparison).

Records failing validation MUST be rejected.

## 5. UAID Reconstruction and Binding (Normative)

For each valid TXT candidate, resolver MUST reconstruct candidate UAID using HCS-14 parameter order:

```
uaid:{target}:{id};uid={uid};registry={registry};proto={proto};nativeId={nativeId}[;domain={domain}][;src={src}]
```

Resolver MUST then compare reconstructed candidate against input UAID after applying HCS-14 canonical ordering rules.

A candidate is accepted only if:
- `target` and `id` match input UAID target/id, and
- reconstructed UAID equals input UAID under canonical parameter ordering.

If no candidates are accepted, resolver MUST fail with `ERR_UAID_MISMATCH`.

If multiple candidates are accepted, resolver MUST select deterministically by lexicographically smallest reconstructed UAID string.

## 6. Downstream Profile Dispatch (Normative)

For selected candidate:

1. Resolver SHOULD use a deterministic local profile-selection policy based on UAID parameters (for example, `registry`, `proto`, and `target`).
2. Resolver SHOULD select the first supported downstream profile under that policy.
3. If no supported downstream profile is available, resolver MUST fail with `ERR_NO_DOWNSTREAM_PROFILE`.

This profile does not define endpoint extraction. Endpoint resolution and protocol-specific verification are delegated to downstream profiles.

## 7. Verification Levels (Normative)

Resolvers SHOULD support both levels below.

### 7.1 Level 1: DNS UAID Binding Verification (Required)

Resolver MUST:
1. Validate `_uaid` record fields (Section 4.3).
2. Reconstruct and match UAID (Section 5).

### 7.2 Level 2: DNSSEC-Assisted Binding (Optional)

Resolver MAY elevate assurance when DNSSEC validation is successful for `_uaid` response.

If resolver claims Level 2 under this profile, Level 1 MUST pass and DNSSEC validation result MUST be positive.

## 8. Resolver Output Requirements (Normative)

For successful resolution, resolver MUST report:
- selected profile (`hcs-14.profile.uaid-dns-web`);
- verification level (`dns-binding` or `dns-binding-dnssec`);
- reconstructed UAID;
- selected downstream profile (if any).

If downstream resolution is performed, resolver SHOULD include downstream result metadata.

## 9. Error Handling (Normative)

Resolvers MUST return structured errors on failure.

Required error codes:

* `ERR_NOT_APPLICABLE` — profile conditions not met.
* `ERR_NO_DNS_RECORD` — `_uaid` record missing.
* `ERR_INVALID_UAID_DNS_RECORD` — TXT payload malformed/invalid.
* `ERR_UAID_MISMATCH` — TXT UAID fields do not match input UAID.
* `ERR_NO_DOWNSTREAM_PROFILE` — no supported downstream profile available.
* `ERR_DISPATCH_FAILED` — downstream profile dispatch attempted and failed.

## 10. Security Considerations

* DNS responses SHOULD be DNSSEC-validated where available.
* Strict UAID reconstruction and equality checks prevent silent parameter drift.
* Downstream profile selection policy SHOULD be deterministic and auditable.
* This profile alone does not prove endpoint control; endpoint integrity depends on downstream profile behavior.

## 11. Examples (Informative)

### 11.1 Example TXT record

```
_uaid.support-agent.example.com TXT "target=aid; id=7Xt9kPmVnBwQ2rY...; uid=support-agent-v1; registry=example-registry; proto=a2a; nativeId=support-agent.example.com; domain=example.com"
```

### 11.2 Example reconstructed UAID

```
uaid:aid:7Xt9kPmVnBwQ2rY...;uid=support-agent-v1;registry=example-registry;proto=a2a;nativeId=support-agent.example.com;domain=example.com
```
