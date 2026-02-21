---
title: HCS-14 Profile Registry
description: Registry of optional, separately versioned profiles that extend HCS-14 with integration-specific discovery and verification mechanisms.
sidebar_position: 140
---

# HCS-14 Profile Registry

This document defines the official **Profile Registry** for the HCS-14 Universal Agent ID Standard.

HCS-14 core specifies identifier structure, canonicalization, and deterministic generation rules.
**Profiles** extend HCS-14 by defining optional, integration-specific mechanisms for discovery, resolution, and verification.

The registry exists to:
- enable multiple discovery and verification mechanisms to coexist;
- prevent the core standard from privileging any single protocol or ecosystem;
- provide a transparent lifecycle and status model for extensions; and
- give implementers a stable reference for supported profiles.

This registry is **normative** with respect to profile identifiers and status definitions.
Individual profile specifications are normative only for implementations that claim support for them.

---

## 1. Profile Status Definitions

Each profile MUST declare one of the following statuses.

### Experimental
- Early draft or exploratory profile.
- MAY change without backward-compatibility guarantees.
- Suitable for prototyping and early adoption only.

### Draft
- Feature-complete and stable enough for multiple implementations.
- Backward-incompatible changes SHOULD be avoided.
- Security considerations SHOULD be documented.

### Recommended
- Meets objective maturity criteria:
  - at least two independent implementations;
  - published test vectors or interoperability tests;
  - documented security review.
- Safe default choice for general implementations.

### Deprecated
- Retained for backward compatibility.
- SHOULD NOT be used for new implementations.
- MAY be removed in a future major revision of HCS-14.

---

## 2. Profile Identification and Versioning

Each profile MUST define:

- **Profile ID**
  A stable, globally unique identifier.
  Recommended format:
```

hcs-14.profile.<short-name>

```

- **Version**
Semantic versioning (`MAJOR.MINOR.PATCH`).

- **Applies to**
The UAID target(s) covered by the profile (e.g., `uaid:aid:*`, `uaid:did:*`).

### Versioning rules
- Changes that break compatibility MUST increment the major version.
- Minor versions MUST remain backward compatible within the same major version.
- Patch versions MUST contain only clarifications or non-behavioral fixes.

---

## 3. Conformance and Claims

Implementations MAY support zero or more profiles.

If an implementation claims support for a profile:
- it MUST implement all normative requirements in that profile;
- it MUST use the correct Profile ID and version when advertising support; and
- it MUST NOT partially implement a profile while claiming full conformance.

HCS-14 core does not require support for any specific profile.

---

## 4. Registered Profiles

The following profiles are currently registered.

### 4.1 UAID DID Resolution Profile

- **Profile ID:** `hcs-14.profile.uaid-did-resolution`
- **Status:** Experimental
- **Version:** 0.1.0
- **Applies to:** `uaid:did:*`
- **Specification:** [uaid-did-resolution.md](./uaid-did-resolution.md)

**Summary:**
Defines a protocol-neutral resolution and output contract for UAIDs that target an existing W3C DID. Relies entirely on the underlying DID method’s resolution and verification rules and introduces no new cryptographic assumptions.

---

### 4.2 AID Resolution via Web & DNS Profile

- **Profile ID:** `hcs-14.profile.aid-dns-web`
- **Status:** Experimental
- **Version:** 0.1.0
- **Applies to:** `uaid:aid:*` where `nativeId` is an FQDN
- **Specification:** [aid-dns-web.md](./aid-dns-web.md)

**Summary:**
Defines discovery of deterministic AID-based UAIDs via DNS TXT records (`_agent.<domain>`) with optional cryptographic verification using the AID Public Key Authentication (PKA) handshake. Includes explicit fallback behavior to protocol-native discovery when DNS records are absent.

---

### 4.3 UAID Resolution via `_uaid` DNS TXT Profile

- **Profile ID:** `hcs-14.profile.uaid-dns-web`
- **Status:** Experimental
- **Version:** 0.1.0
- **Applies to:** `uaid:aid:*` and `uaid:did:*` where `nativeId` is an FQDN
- **Specification:** [uaid-dns-web.md](./uaid-dns-web.md)

**Summary:**
Defines DNS TXT discovery at `_uaid.<nativeId>` where the record directly carries HCS-14 identifier components and routing parameters. Resolvers reconstruct and match UAID deterministically, then dispatch to downstream resolution profiles.

---

## 5. Adding New Profiles

New profiles MAY be added through the HCS improvement or specification process.

A new profile submission SHOULD include:
- a unique Profile ID;
- a clearly scoped applicability statement;
- normative discovery and/or verification rules;
- security considerations;
- at least one illustrative example or test vector.

Profiles SHOULD be designed to:
- avoid assumptions about global precedence;
- operate independently of other profiles; and
- degrade gracefully when unsupported.

---

## 6. Neutrality and Non-Endorsement

Inclusion in this registry does **not** imply endorsement of any protocol, registry, or ecosystem.

The registry:
- does not rank profiles;
- does not mandate default selection;
- does not require resolvers to prefer one profile over another.

Resolver policy, profile selection, and precedence are implementation-defined unless explicitly stated within a profile.

---

## 7. Change Log

- **0.1.0** — Initial registry with UAID DID Resolution and AID Web/DNS profiles.
- **0.2.0** — Added UAID DNS TXT profile (`hcs-14.profile.uaid-dns-web`).

---

## 8. References (Informative)

- HCS-14 Universal Agent ID Standard (core)
- W3C Decentralized Identifier (DID) Core
- RFC 2119: Key words for use in RFCs to Indicate Requirement Levels
```
