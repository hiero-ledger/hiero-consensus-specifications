---
title: HCS-14 Profile - UAID DID Resolution
description: Profile for resolving uaid:did identifiers to DID Documents and service endpoints in a protocol-neutral way.
sidebar_position: 141
---

# HCS-14 Profile: UAID DID Resolution

**Profile ID:** `hcs-14.profile.uaid-did-resolution`  
**Status:** Experimental  
**Version:** 0.1.0  
**Applies to:** `uaid:did:*`

## 1. Scope

This profile defines a minimal, protocol-neutral resolution and output contract for **UAID identifiers that target an existing W3C DID** (i.e., `uaid:did:*`). It specifies:

- how to interpret a `uaid:did` identifier for the purpose of resolution,
- what a conformant resolver MUST return as a DID Document (or DID Document–compatible representation),
- how a resolver SHOULD map UAID routing parameters into a DID Document `service` section (when helpful), and
- how to represent verification status and errors.

This profile does **not** define any new DID method, key format, cryptographic suite, or trust framework. It relies on the underlying base DID method’s verification and resolution rules.

## 2. Terminology and Conformance

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119.

A resolver is **conformant** with this profile if it implements all requirements in Sections 3–6.

## 3. Inputs and UAID Interpretation (Normative)

### 3.1 Supported identifiers

A conformant resolver MUST accept identifiers of the form:

```

uaid:did:{id};{parameters}

````

Where:
- `{id}` is the UAID identifier component derived from the subject’s base DID method-specific identifier (sanitized by HCS-14 core rules).
- `{parameters}` is a semicolon-separated list of key-value parameters as defined in HCS-14 core (e.g., `uid`, `registry`, `proto`, `nativeId`, `domain`, `src`).

### 3.2 Parameter handling

A conformant resolver MUST:
- parse parameters as an unordered map for processing, while preserving the original string for display/debug if needed;
- ignore unknown parameters (forward compatibility), unless the resolver is operating in a strict mode explicitly configured by the operator.

A conformant resolver MUST NOT treat routing parameters as authoritative identity claims. Parameters are routing hints and correlation metadata; identity verification is performed via DID resolution and verification of the underlying base DID.

### 3.3 Base DID recovery using `src`

Because `uaid:did:{id}` may be derived from a sanitized base DID identifier, resolvers often cannot reconstruct the exact original DID from `{id}` alone.

If the UAID includes a `src` parameter, the resolver MUST:
- decode `src` as multibase base58btc (prefix `z...`), yielding the UTF-8 string of the full original base DID; and
- treat the decoded DID string as the **base DID** for the purposes of resolution.

If the UAID does not include `src`, the resolver MAY still resolve using `{id}` if it can be unambiguously interpreted as a DID method-specific identifier in the resolver’s environment. If the resolver cannot unambiguously determine a base DID, it MUST return a resolution error as described in Section 6.

> Note: HCS-14 core defines when and how `src` is included. This profile specifies how to use it when present.

## 4. Resolution Procedure (Normative)

### 4.1 Resolution algorithm

A conformant resolver MUST perform the following steps:

1. **Parse** the UAID and parameters.
2. **Determine base DID**:
   - If `src` is present, recover the base DID per Section 3.3.
   - Otherwise, attempt to infer the base DID from `{id}` using implementation-defined rules.
3. **Resolve base DID** using a DID resolver consistent with W3C DID Resolution for that DID method.
4. **Validate output** per Section 5.
5. **Construct the response** (DID Document + metadata) per Section 5 and Section 6.

### 4.2 Caching

Resolvers MAY cache successful resolutions. If caching is used, resolvers SHOULD:
- respect any cache-control or TTL semantics provided by the underlying DID method; and
- expose whether a response was served from cache in resolver metadata.

Resolvers MUST NOT cache resolution failures longer than successful results unless an operator explicitly configures negative caching.

## 5. Output Requirements (Normative)

### 5.1 Minimal DID Document requirements

For a successful resolution, a conformant resolver MUST return a DID Document (or DID Document–compatible JSON object) that includes:

- `id`: the input UAID string exactly as provided (the subject identifier in this profile is the UAID).
- `alsoKnownAs`: an array including the resolved base DID string (when known).
- `verificationMethod`: zero or more entries representing verification material if present in the base DID Document.
- `authentication` and/or `assertionMethod`: relationships consistent with the base DID method’s document, when present.

The resolver MUST NOT invent cryptographic verification methods. If the base DID Document contains no keys or verification relationships, the resolver MUST omit them rather than synthesize new ones.

### 5.2 Service endpoint mapping

Resolvers MAY include `service` entries derived from:
- service entries present in the base DID Document; and/or
- routing parameters from the UAID (e.g., `proto`, `nativeId`, `domain`) when the resolver can produce a meaningful, non-deceptive mapping.

If a resolver includes `service` entries derived from UAID parameters, it MUST:
- clearly distinguish (via `service` `type` or resolver metadata) whether the entry is **copied from the base DID Document** or **derived from UAID hints**;
- ensure derived entries do not claim stronger trust than warranted (e.g., do not mark them as authenticated if they are merely hints).

Recommended conventions:
- For derived services, use a `type` suffix or prefix such as `HintedService`, or include an additional object field such as `source: "uaid-parameters"`.

### 5.3 Protocol neutrality

A resolver conformant with this profile MUST NOT require any specific protocol integration (e.g., DNS discovery, well-known endpoints, registry APIs). Those mechanisms belong in separate profiles.

If the base DID method itself implies a resolution mechanism (e.g., `did:web` over HTTPS), the resolver may implement it as part of DID resolution. This is not considered “choosing a winner” because it is intrinsic to the base DID method selected by the subject.

## 6. Verification and Errors (Normative)

### 6.1 Verification metadata

A resolver conformant with this profile MUST produce resolver metadata that indicates:

- whether the base DID was successfully resolved;
- whether the returned DID Document keys/relationships were validated according to the base DID method rules (if applicable);
- whether any `service` entries were copied vs derived.

A minimal metadata structure is:

```json
{
  "resolved": true,
  "baseDid": "did:example:123",
  "baseDidResolved": true,
  "verification": {
    "method": "did-resolution",
    "assurance": "base-method",
    "details": "Validated according to base DID method rules"
  },
  "services": {
    "copiedFromBaseDidDocument": true,
    "derivedFromUaidParameters": false
  }
}
````

This schema is illustrative; implementations MAY extend it, but MUST expose equivalent information.

### 6.2 Error cases

If resolution fails, a conformant resolver MUST return:

* `resolved: false`
* an `error` object with:

  * `code`: a stable machine-readable string,
  * `message`: a human-readable explanation,
  * `details`: optional structured data.

Required error codes:

* `ERR_INVALID_UAID` — UAID parsing failed or scheme is not `uaid:did`.
* `ERR_BASE_DID_UNDETERMINED` — base DID could not be reconstructed or inferred.
* `ERR_DID_RESOLUTION_FAILED` — base DID resolution failed (network, method, or document error).
* `ERR_DID_DOCUMENT_INVALID` — base DID document did not meet minimal validity checks (e.g., missing `id` or malformed JSON).

Example error response (illustrative):

```json
{
  "resolved": false,
  "error": {
    "code": "ERR_BASE_DID_UNDETERMINED",
    "message": "Unable to determine base DID; provide src parameter or configure method inference.",
    "details": {
      "uaid": "uaid:did:abc123;uid=0;registry=self"
    }
  }
}
```

## 7. Security Considerations

* **Parameter trust**: UAID parameters are hints. Treat them as untrusted input unless corroborated by the base DID Document or higher-assurance profiles.
* **Service injection**: Do not elevate derived services to trusted endpoints without additional verification. Prefer copying `service` entries directly from the base DID Document where possible.
* **`src` integrity**: `src` can reconstruct the original DID string, but it is not itself a proof. Any DID method–specific security must be enforced by the base DID resolution and verification process.
* **Privacy**: Including `alsoKnownAs` and base DID strings can increase correlation risk. Implementations MAY provide a privacy mode that omits `alsoKnownAs` unless explicitly requested by the client.

## 8. Examples (Informative)

### 8.1 Basic `uaid:did` resolution with `src`

**Input UAID:**

```
uaid:did:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK;uid=0;registry=self;proto=hcs-10;nativeId=hedera:testnet:0.0.123456;src=z6469643a6b65793a7a364d6b68615867425a44766f74446b4c353235376661697a74694769433251744b4c4770626e6e4547746132646f4b
```

**Notes:**

* `src` decodes to the full base DID string (illustrative encoding shown).

**Illustrative output (truncated):**

```json
{
  "didDocument": {
    "id": "uaid:did:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK;uid=0;registry=self;proto=hcs-10;nativeId=hedera:testnet:0.0.123456;src=...",
    "alsoKnownAs": ["did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK"],
    "verificationMethod": [],
    "service": []
  },
  "metadata": {
    "resolved": true,
    "baseDidResolved": true,
    "verification": {
      "method": "did-resolution",
      "assurance": "base-method"
    },
    "services": {
      "copiedFromBaseDidDocument": false,
      "derivedFromUaidParameters": false
    }
  }
}
```

### 8.2 Deriving a service entry from UAID parameters (hinted)

If the base DID Document contains no `service` entries and the resolver chooses to provide a hinted entry, it might emit:

```json
{
  "service": [
    {
      "id": "#hcs14-hinted-service-1",
      "type": "HintedService",
      "serviceEndpoint": {
        "proto": "hcs-10",
        "nativeId": "hedera:testnet:0.0.123456"
      },
      "source": "uaid-parameters"
    }
  ]
}
```

This is not a verified endpoint; it is a structured hint.

## 9. References (Informative)

* HCS-14 core specification (Universal Agent ID Standard)
* W3C DID Core and DID Resolution (for base DID interpretation and resolution semantics)

