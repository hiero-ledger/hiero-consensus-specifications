---
title: "HCS-24: Decentralized Identity Trust Registry"
description: This specification provides a standard way to define, register, and reference trust records for decentralized identity use cases, enabling trust models relying on secure and efficient Hedera consensus.
sidebar_position: 24
---

# HCS-24 Standard: Decentralized Identity Trust Registry  


### Status: Draft

### Table of Contents

- [HCS-24 Standard: Decentralized Identity Trust Registry](#hcs-24-standard-decentralized-identity-trust-registry)
  - [Status: Draft](#status-draft)
  - [Table of Contents](#table-of-contents)
  - [Authors](#authors)
  - [Abstract](#abstract)
  - [Motivation](#motivation)
  - [Specification](#specification)
    - [Architecture Overview](#architecture-overview)
    - [Trust Record Definition](#trust-record-definition)
      - [Recommendations on jurisdiction priority](#recommendations-on-jurisdiction-priority)
      - [Trust Record Examples](#trust-record-examples)
    - [Trust Record Topics](#trust-record-topics)
    - [Trust Registry Topics](#trust-registry-topics)
      - [Registry Creation](#registry-creation)
      - [Trust Registry Operations](#trust-registry-operations)
        - [Register Trust Record](#register-trust-record)
        - [Revoke Trust Record](#revoke-trust-record)
    - [Trust Records Referencing](#trust-records-referencing)
      - [HRL Format](#hrl-format)
    - [Credential Schemas and HCS-13](#credential-schemas-and-hcs-13)
    - [Governance](#governance)
    - [Security Considerations](#security-considerations)
      - [1. Topic Access Control](#1-topic-access-control)
      - [2. Trust Record Verification](#2-trust-record-verification)
      - [3. Resolver Trust Model](#3-resolver-trust-model)
      - [4. Data Privacy](#4-data-privacy)
      - [5. Availability and DoS](#5-availability-and-dos)
      - [6. Key Management](#6-key-management)
    - [TRQP Integration (Appendix A)](#trqp-integration-appendix-a)
      - [Conceptual Mapping](#conceptual-mapping)
      - [Resolver Implementation](#resolver-implementation)
      - [Query Mapping Example](#query-mapping-example)
    - [Conclusion](#conclusion)
    - [References](#references)

## Authors

- AlexanderShenshin - https://github.com/AlexanderShenshin

## Abstract

This specification builds upon [HCS-2](hcs-2.md) and provides a standard way to define, register, and reference trust records for decentralized identity use cases, enabling trust models relying on secure and efficient Hedera consensus.

## Motivation

The main goal is to address the need for reliable Trust Registry options for Decentralized Identity / SSI ecosystems, where intra- and inter-ecosystem trust model becomes a critical factor.

Hedera and, specifically, Hedera Consensus Service (HCS) provide a reliable and efficient way to store and manage trust records.

Key motivations include:

1. Providing a standardized approach for HCS-based decentralized identity Trust Registries
2. Addressing the need for a reliable Trust Registry for Decentralized Identity / SSI space
3. Extending Hiero Decentralized Identity use cases with HCS-based Trust Registry that can be used independently of other parts of the ecosystem (Hedera DID, AnonCreds)
4. Leveraging existing HCS infrastructure and ecosystem, including HCS-2: Topic Registries as a base

## Specification

This specification defines a domain-specific framework for creation and management of **Trust Registries** in Decentralized Identity / SSI ecosystems.  

### Architecture Overview

The HCS-24 standard defines two types of topics:

1. **Trust Record Topics**: Topics that store Trust Records as HCS-1 files. Each record is stored in an HCS-1 topic with versioning being managed through HCS-2 operations.
2. **Trust Registry Topics**: An HCS-2 registry that enables discoverability of trust records across the ecosystem.

This architecture provides separation of concerns while enabling essential trust model and discovery capabilities.

### Trust Record Definition

HCS-24 defines a **Trust Record** as a JSON object that contains DID and other relevant information about the trust subject depending on a specific type.

Record providing information on a subject issuer/verifier that should be considered trusted by parties relying on a specific Trust Registry instance.

A valid trust record MUST comply with the following schema:

| Property        | Type     | Required | Description                                                                                                                                                              |
|-----------------|----------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `type`          | string   | Yes      | Trust record type, value MUST be either "issuer" or "verifier"                                                                                                           |
| `did`           | string   | Yes      | The Decentralized Identifier (DID) of the trusted party                                                                                                                  |
| `name`          | string   | Yes      | A human-readable name of the issuer/verifier party                                                                                                                       |
| `description`   | string   | No       | A human-readable description of the issuer/verifier party                                                                                                                |
| `urls`          | string[] | No       | A list of trusted base URLs associated with the issuer/verifier party                                                                                                    |
| `tags`          | string[] | No       | A list of tags assigned to the party within the ecosystem (e.g., `University`). Specific use cases should be defined by ecosystem/registry governance                    |
| `jurisdictions` | string[] | No       | A list of regions or jurisdictions where the party is recognized (e.g., `EU`, `US`). Depending on registry governance, these can be defined in registry metadata instead |
| `policyPointer` | string   | No       | A URI pointing to a specific policy or legal agreement governing this party inclusion                                                                                    |

#### Recommendations on jurisdiction priority

It is recommended that if Trust Registry metadata defines a `jurisdictions` list, the issuer's jurisdiction is determined in a following way:
- If registry metadata defines single jurisdiction, it MUST be considered the issuer's jurisdiction. If the issuer record defines different jurisdiction, it MUST be ignored.
- If registry metadata defines multiple jurisdictions, the following recommendations apply:
    1. If the issuer jurisdiction list is a proper subset of the registry's jurisdictions, the list from issuer record MUST be considered as is.
    2. Jurisdiction entries in issuer record that are not included in the jurisdictions list in registry metadata MUST be ignored.

Please note that, as stated before, the governance policy of the trust registry is out of scope for this specification and should be defined by ecosystem governance bodies.

#### Trust Record Examples

**Issuer Trust Record:**

```json
{
    "type": "issuer",
    "did": "did:hedera:testnet:23g2MabDNq3KyB7oeH9yYZsJTRVeQ24DqX8o6scB98e3_0.0.5217215",
    "name": "University of Porto",
    "description": "Official issuer service of University of Porto, responsible for issuing credentials related to education and research.",
    "tags": ["University"],
    "jurisdictions": ["EU"]
}
```

**Verifier Trust Record:**

```json
{
    "type": "verifier",
    "did": "did:hedera:testnet:FAeKMsqnNc2bwEsC8oqENBvGqjpGu9tpUi3VWaFEBXBo_0.0.5896419",
    "name": "Demo DeFi Platform",
    "description": "Official verifier service of Demo DeFi Platform (experimental)",
    "urls": ["https://defiplatform.verifier.org/"],
    "tags": ["DeFi", "DEX"],
    "jurisdictions": ["EU", "US"]
}
```

### Trust Record Topics

Trust Record topics store the actual record JSON objects as HCS-1 files.

The workflow for creating and managing Trust Record topic is:

1. Create an HCS-1 file containing the record data in JSON format
2. Create an HCS-2 topic to manage versions of the trust record
3. Register the HCS-1 file in the HCS-2 topic using the register operation

This approach leverages:

- HCS-1 for storing the trust record content (which may exceed the 1KB message limit for certain configurations)
- HCS-2 for versioning and managing references to the trust record

Trust record properties **MAY** be partially included in HCS-2 message metadata.
It's recommended to only include properties relevant for a discovery process (e.g., `type`, `did`, `name`).

Example trust record registration in an HCS-2 topic:

```json
{
  "p": "hcs-24",
  "op": "register",
  "t_id": "0.0.123456", // Topic ID of the HCS-1 file containing the trust record
  "metadata": {
    "type": "issuer",
    "did": "did:hedera:testnet:23g2MabDNq3KyB7oeH9yYZsJTRVeQ24DqX8o6scB98e3_0.0.5217215",
    "name": "University of Porto"
  },
  "m": "Registration of University of Porto trust record"
}
```

### Trust Registry Topics

HCS-24 Trust Registry is an HCS-2 guarded registry that enables discoverability of trust records across the ecosystem.
Trust Registry HCS topics use a modified HCS-2 format with a memo that can optionally reference registry metadata (published as HCS-1 file).

#### Registry Creation

A message registry topic is created as an HCS topic with a specific memo format that indicates its purpose and configuration:

`hcs-24:[indexed]:[ttl]:[registryMetadataTopicId]`

| Field                     | Description                                                                            | Example Value |
|---------------------------|----------------------------------------------------------------------------------------|---------------|
| `hcs-24`                  | Indicates this is a message registry following the HCS-24 standard                     | `hcs-24`      |
| `indexed`                 | Enum value (0 or 1) indicating if all messages need to be processed or only the latest | `0`           |
| `ttl`                     | Time-to-live in seconds for caching purposes                                           | `86400`       |
| `registryMetadataTopicId` | Optional Topic ID pointing to HCS-1 file with trust registry metadata                  | `0.0.123456`  |

Example memo: `hcs-24:0:86400:0.0.123456`

#### Trust Registry Operations

The trust registry supports operations for managing trust records discoverability.

##### Register Trust Record

To register a trust record in the registry:

```json
{
  "p": "hcs-24",
  "op": "register",
  "t_id": "0.0.123456", // Topic ID of the HCS-1 file containing the trust record
  "metadata": {
    "type": "issuer",
    "did": "did:hedera:testnet:23g2MabDNq3KyB7oeH9yYZsJTRVeQ24DqX8o6scB98e3_0.0.5217215",
    "name": "University of Porto"
  },
  "m": "Registration of University of Porto trust record"
}
```

##### Revoke Trust Record

To mark a schema as deprecated in the discovery registry:

```json
{
  "p": "hcs-24",
  "op": "revoke",
  "t_id": "0.0.123456", // Topic ID of the HCS-1 file containing the trust record
  "metadata": {
    "reason": "Issuer has stopped operating in the ecosystem"
  },
  "m": "Issuer has been revoked due to inactivity"
}
```

### Trust Records Referencing

Trust records can be referenced in various contexts using Hedera Resource Locators (HRLs).

#### HRL Format

The HRL format for referencing a trust records follows the pattern:

`hcs://13/[topicId]#[sequenceNumber]`

Where `topicId` is the topic ID of the Trust Registry topic and `sequenceNumber` is the sequence number of the specific trust record.

Example: `hcs://13/0.0.123456#42`

### Credential Schemas and HCS-13

While HCS-24 operates with the trust in actors, it does not define the format of the credentials themselves.
For defining and registering credential schemas, it is recommended to use the [HCS-13 Standard: Schema Registry and Validation](./hcs-13.md). 

HCS-13 provides a robust mechanism for:
- Storing JSON Schema definitions using HCS-1.
- Versioning schemas via HCS-2.
- Referencing schemas in messages and NFTs using HRLs (e.g., `hcs://13/{topicId}`).

By leveraging HCS-13, ecosystems using HCS-24 can ensure that credentials are not only issued by trusted parties but also conform to standardized, machine-readable formats.


### Governance

While HCS-24 provides the technical mechanism for recording and discovering trust data, the actual **Governance Framework** that defines the rules for inclusion, revocation, and operational policies is a responsibility of the specific ecosystem and is not covered in this specification.

Key governance considerations that are out of scope for HCS-24 but essential for a functioning trust registry include:

* **Operational Policies**: Defining the criteria and processes for onboarding/offboarding trusted parties.
* **Authorization**: Determining which entities have the right to submit, update, or revoke entries in a specific registry topic (typically managed via Hedera Topic admin/submit keys).
* **Legal Frameworks**: Any legal agreements or jurisdictions that govern the trust relationships within the ecosystem.
* **Registry Metadata**: If a governance framework is established, it SHOULD be referenced or published via registry metadata (e.g., as an HCS-1 file) to ensure transparency for all ecosystem participants.

This separation of concerns allows HCS-24 to remain a neutral technical building block that can support a wide variety of governance models, from strictly centralized to fully decentralized or federated approaches.


### Security Considerations

The HCS-24 standard relies on the security properties of the Hedera Consensus Service (HCS) and the Trust over IP (ToIP) stack. Implementers should consider the following security aspects:

#### 1. Topic Access Control

The security of an HCS-24 registry is primarily governed by the Hedera Topic's keys:
* **Admin Key**: Controls the ability to update topic metadata or delete the topic. This key SHOULD be held by a multi-signature account or a decentralized governance body.
* **Submit Key**: Controls who can register or revoke trust records. For public registries, this key SHOULD be restricted to authorized ecosystem actors.
* **No Submit Key**: If the topic is created without a submit key, anyone can submit messages, which is NOT RECOMMENDED for a trust registry as it enables spam and unauthorized entries.

#### 2. Trust Record Verification

Clients and resolvers MUST NOT rely solely on the presence of a record in the registry:
* **Integrity**: When resolving records via HCS-1, the integrity of the content MUST be verified against the hash or consensus state provided by the Hedera network.
* **HRL Resolution**: When following HRLs, ensure that the `topicId` matches the expected registry to prevent "registry swapping" attacks.

#### 3. Resolver Trust Model

In the HCS-24 architecture, resolvers are considered **untrusted infrastructure**:
* Resolvers provide a convenience API (like TRQP), but the source of truth is always the Hedera Ledger.
* High-security applications SHOULD run their own resolvers or verify resolver outputs against the ledger directly.
* Resolvers MUST correctly handle revocations. A resolver that fails to index a `revoke` operation may provide outdated and incorrect trust status.

#### 4. Data Privacy

* **PII on Ledger**: Trust records are public. Ecosystems MUST ensure that no personally identifiable information (PII) is included in trust records unless it is intended to be public (e.g., a corporate name).
* **Metadata Leakage**: Even if record content is encrypted (not recommended for public registries), the timing and frequency of updates might leak information about ecosystem activities.

#### 5. Availability and DoS

* **Hedera Resilience**: HCS-24 inherits the high availability and DDoS resistance of the Hedera network.
* **Spam Prevention**: Registry administrators should monitor for spam messages. The cost of HCS messages provides a natural economic barrier, but additional filtering at the resolver level may be necessary.

#### 6. Key Management

* The private keys for the Registry Topic (admin/submit) are critical assets. If compromised, an attacker could revoke all trusted parties or register malicious ones.
* It is RECOMMENDED to use hardware security modules (HSM) or multi-party computation (MPC) for managing these keys.


### TRQP Integration

> Optional integration for resolvers implementing ToIP TRQP

The Trust Registry Query Protocol (TRQP) defined by the Trust over IP (ToIP) Foundation provides a standardized way to query trust registries.
HCS-24 registries can be exposed via TRQP-compliant resolvers by mapping HCS-24 data structures to TRQP concepts.

#### Conceptual Mapping

| TRQP Concept        | HCS-24 Implementation                                                              |
|---------------------|------------------------------------------------------------------------------------|
| Governing Authority | The entity/account(s) holding the admin/submit keys for the HCS-24 Registry Topic. |
| Trust Registry      | An HCS-24 Registry Topic (identified by its Hedera Topic ID).                      |
| Trust Record        | An HCS-24 entry (stored as an HCS-1 file and registered in the HCS-24 topic).      |
| Trust Subject       | The `did` specified in the HCS-24 trust record.                                    |
| Role                | Mapped from HCS-24 `type` (`issuer` or `verifier`).                                |

#### Resolver Implementation

A TRQP-compliant resolver for HCS-24 SHOULD:

1. **Monitor the HCS-24 Topic**: Index all messages in the designated HCS-24 Registry Topic.
2. **Resolve Trust Records**: For each `register` operation, fetch the corresponding HCS-1 file to obtain the full trust record metadata.
3. **Handle Revocations**: Update the local state when a `revoke` operation is encountered for a specific `t_id`.
4. **Expose TRQP Interface**: Implement the TRQP query API (e.g., `is-trusted`) using the indexed data.

#### Query Mapping Example

When a TRQP `is-trusted` query is received:

- **Input**: `did`, `role`, `credential_type` (optional), `jurisdiction` (optional)
- **HCS-24 Lookup**:
    1. Find the latest non-revoked record in the registry where `record.did == query.did` and `record.type == query.role`.
    2. If `jurisdiction` is provided in the query, verify it matches the record's `jurisdictions` (considering the recommendations on jurisdiction priority defined in the Specification section).
    3. If `credential_type` is provided, it can be matched against the record's `tags` or metadata (if the ecosystem adopts such a convention).
- **Output**: Return the trust status and associated metadata as defined by TRQP.

By implementing this mapping, HCS-24 registries can participate in broader ToIP-compliant trust ecosystems while leveraging Hedera's decentralized consensus for the underlying data integrity.

### Conclusion

The HCS-24 standard provides a robust and standardized framework for building decentralized identity trust registries on the Hedera network.
By leveraging HCS-1 for record storage and HCS-2 for efficient discovery, it helps to create a scalable and reliable trust infrastructure.

Key benefits of the HCS-24 standard include:

1. **Standardized Trust Records**: Provides a consistent format for defining and registering trusted issuers and verifiers.
2. **Decentralized Integrity**: Utilizes Hedera Consensus Service to ensure the tamper-proof nature of trust data.
3. **Interoperability**: Enables cross-ecosystem trust through standardized HRL references and TRQP integration.
4. **Separation of Concerns**: Maintains a clear distinction between trust record metadata and the registry's discovery mechanism.
5. **Ecosystem Leverage**: Builds upon existing HCS-1, HCS-2, and HCS-13 standards to provide a comprehensive identity solution.

By adopting this standard, ecosystems can establish reliable, transparent, and machine-readable trust models that are essential for the widespread adoption of Decentralized Identity and SSI solutions.


### References

### Normative
- [HCS-2: Topic Registries](./hcs-2.md)

### Informative
- [DID](https://www.w3.org/TR/did-core/)
- [VC](https://www.w3.org/TR/vc-data-model/)
- [TRQP](https://github.com/trustoverip/concepts-and-terminology-wg/blob/master/concepts/trust-registry-query-protocol.md)

### License

This document is licensed under [Apache‑2.0](https://www.apache.org/licenses/LICENSE-2.0).
