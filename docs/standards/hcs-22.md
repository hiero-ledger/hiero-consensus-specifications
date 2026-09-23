# **HCS-22: Identity Binding Events**

## 1. Standard Name
**HCS-22 — Identity Binding Events**

---

## 2. Category
**Identity / Application-Layer Primitive**

---

## 3. Abstract
This standard defines a canonical Hedera Consensus Service (HCS) protocol for expressing identity binding assertions using HCS-4 message structure.

Identity Binding Events enable participants or authorized issuers to assert verifiable relationships between identities (e.g., Hedera account IDs, DIDs, public keys, or external addresses) without prescribing authentication, custody, or authorization semantics.

These bindings produce neutral, append-only identity facts consumable by downstream systems such as wallets, messaging layers, trust networks, compliance tooling, and research environments.

The standard is intentionally minimal, declarative, and non-custodial. It does not define identity issuance, authentication flows, revocation enforcement, or trust interpretation logic.

---

## 4. Motivation
Decentralized applications frequently require the ability to associate multiple identifiers with a single actor. Today, these associations are commonly managed through proprietary databases, centralized resolvers, or implicit wallet assumptions, limiting interoperability and auditability.

Hedera Consensus Service provides a neutral substrate for publishing identity binding assertions as shared, verifiable events. By standardizing a minimal binding primitive aligned with HCS-4, applications can resolve identity relationships without coupling to specific wallet implementations, authentication systems, or trust frameworks.

This standard separates **identity assertion** from **identity authority**, **authentication**, and **authorization**.

---

## 5. Specification (Normative)

### 5.1 Protocol Identifier
p: "hcs-22"

---

### 5.2 Operations

bind  
Create a new identity binding assertion.

---

### 5.3 Message Structure (HCS-4 Aligned)

{
  "p": "hcs-22",
  "op": "bind",
  "t_id": "0.0.topicId",
  "data": {
    "subject": "did:hedera:0.0.x",
    "bound_identifier": "string",
    "binding_type": "string",
    "issuer": "did:hedera:0.0.y",
    "confidence": "self_asserted | issuer_attested",
    "scope": "string",
    "context": {
      "world_id": "string",
      "reason": "string"
    }
  }
}

---

### 5.4 Field Definitions

| Field              | Description                                                                    |
| ------------------ | ------------------------------------------------------------------------------ |
| `p`                | Protocol identifier (`hcs-22`)                                                 |
| `op`               | Operation (`bind`)                                                             |
| `t_id`             | Topic ID where the event is published                                          |
| `data.subject`     | Primary decentralized identifier being described                               |
| `data.bound_identifier` | Identifier being bound to the subject                                    |
| `data.binding_type`     | Type of binding (e.g., `hedera_account`, `evm_address`, `public_key`, `email`) |
| `data.issuer`      | DID asserting the binding                                                      |
| `data.confidence`  | Declarative confidence level of binding                                        |
| `data.scope`       | Optional application or domain scope                                           |
| `data.context`     | Optional application-specific metadata                                         |

---

### 5.5 Normative Requirements

- Identity Binding Events **MUST** conform to HCS-4 message structure.
- Identity Binding Events **MUST** be append-only.
- The `issuer` **MUST** be an identifiable entity.
- This standard **MUST NOT** define authentication or authorization semantics.
- This standard **MUST NOT** define enforcement, revocation propagation, or resolution priority rules.
- Consumers **MUST** treat bindings as assertions, not guarantees.

---

## 6. Design Principles

- **Neutrality**: No assumptions about identity authority or trust.
- **Minimalism**: Simple binding assertions only.
- **Composability**: Bindings usable across protocols and applications.
- **Auditability**: All bindings are time-ordered, immutable events.
- **Separation of Concerns**: Binding ≠ authentication ≠ authorization.

---

## 7. Security Considerations

- Binding assertions may be false or malicious.
- Confidence levels are declarative and not enforced.
- Consuming applications are responsible for validation policies.
- World or topic isolation is recommended for bounded deployments.

---

## 8. Interoperability

- Compatible with HCS-1, HCS-2, and HCS-4 JSON message conventions.
- May be combined with other standards such as Trust Allocation Events or capability-based authorization.
- This standard is designed to compose cleanly with **HCS-23 (Trust Allocation Events)**, which may consume identity bindings as inputs to trust signaling and policy evaluation.
- Fully consumable via Hedera Mirror Nodes without specialized indexing.

---

## 9. Reference Implementation (Non-Normative)

A reference implementation exists demonstrating identity bindings used to resolve messaging and trust relationships across heterogeneous address formats. This implementation is informational and does not define standard behavior.

---

## 10. Backward Compatibility

This standard is purely additive and introduces no breaking changes to existing HCS topics or message schemas.

---

## 11. Governance and Evolution

Future extensions may include revocation semantics, binding expiry, or confidence scoring. These features are intentionally excluded from version 1.0 to preserve neutrality and simplicity.

---

## 12. Acknowledgements

This proposal aligns with the mission of Hashgraph Online to promote open, composable, and interoperable identity primitives for the Hedera ecosystem.

---

### Status: **Proposed**
