# **HCS-23: Trust Allocation Events**

## 1. Standard Name

**HCS-TA — Trust Allocation Events**

---

## 2. Category

**Application-Layer Primitive**

---

## 3. Abstract

This standard defines a canonical Hedera Consensus Service (HCS) message format for representing explicit trust allocation between decentralized identities.

Trust Allocation Events enable participants to allocate a limited unit of trust to another participant, producing verifiable trust facts that can be consumed deterministically by downstream systems. These trust facts may be used by applications for messaging policy, access control, governance, moderation, payments gating, and research without prescribing interpretation logic.

The standard is intentionally minimal, append-only, non-financial, and neutral. It does not define identity binding, reputation scoring, aggregation, decay, revocation, or enforcement semantics. Interpretation of trust signals is explicitly left to consuming applications.

---

## 4. Motivation

Trust is a foundational input to many decentralized applications, yet it is typically modeled implicitly, enforced off-chain, or embedded within platform-specific logic. Identity alone does not provide sufficient context for authorization, coordination, or policy decisions.

Applications repeatedly re-implement trust logic using proprietary databases or opaque heuristics, reducing interoperability and auditability.

Hedera Consensus Service provides an opportunity to represent trust signals as shared, verifiable, and neutral ledger events. By standardizing a minimal trust allocation primitive, ecosystems can exchange trust facts without coupling to specific reputation models, scoring systems, or governance frameworks.

This standard separates **trust signaling** from **trust interpretation**.

---

## 5. Specification (Normative)

### 5.1 Event Type

```
event_type: "trust_allocation"
```

---

### 5.2 Required Fields

```json
{
  "standard": "HCS-TA",
  "version": "1.0",
  "event_type": "trust_allocation",
  "allocator": "did:hedera:0.0.x",
  "recipient": "did:hedera:0.0.y",
  "unit": 1,
  "timestamp": 1734220800
}
```

---

### 5.3 Optional Fields

```json
{
  "constraints": {
    "max_active_allocations": 9,
    "exclusive": true
  },
  "context": {
    "world_id": "string",
    "reason": "string"
  }
}
```

---

### 5.4 Field Definitions

| Field         | Description                                |
| ------------- | ------------------------------------------ |
| `standard`    | Identifier for this standard               |
| `version`     | Standard version                           |
| `event_type`  | MUST be `"trust_allocation"`               |
| `allocator`   | DID of the entity allocating trust         |
| `recipient`   | DID of the entity receiving trust          |
| `unit`        | Integer representing a single trust unit   |
| `timestamp`   | Unix timestamp of allocation               |
| `constraints` | Declarative limits on allocation semantics |
| `context`     | Optional application-specific metadata     |

---

### 5.5 Normative Requirements

* Allocators **MUST** be identifiable entities.
* Trust Allocation Events **MUST** be append-only.
* Trust Allocation Events **MUST NOT** represent financial value.
* This standard **MUST NOT** define identity binding, scoring, aggregation, revocation, or decay semantics.
* Interpretation of trust signals **MUST NOT** be defined by this standard.

---

## 6. Design Principles

* **Minimalism**: Single event type with limited required fields.
* **Neutrality**: No assumptions about application behavior.
* **Scarcity by Convention**: Limits expressed declaratively, not enforced at protocol level.
* **Composability**: Trust facts usable across domains and systems.
* **Determinism**: Identical ledger state yields identical trust facts.

---

## 7. Security Considerations

* Sybil resistance mechanisms are out of scope.
* Abuse mitigation is delegated to consuming applications.
* World or topic isolation is recommended for bounded deployments.
* Events are non-custodial and non-financial, reducing regulatory risk.

---

## 8. Interoperability

* Compatible with HCS-1 and HCS-2 JSON message conventions.
* Identity resolution and binding are intentionally out of scope and may be defined by separate standards.
* This standard is designed to compose cleanly with **HCS-IB (Identity Binding Events)**, which may be used by consuming applications to resolve identifiers referenced in Trust Allocation Events.
* Fully consumable via Hedera Mirror Nodes without custom indexing logic.

---

## 9. Reference Implementation (Non-Normative)

A reference implementation exists demonstrating Trust Allocation Events used as deterministic inputs to messaging and payment policy decisions. This implementation is informational and does not define standard behavior.

---

## 10. Backward Compatibility

This standard is purely additive and introduces no breaking changes to existing HCS topics or message schemas.

---

## 11. Governance and Evolution

Future extensions may include revocation, delegation, or decay semantics. Such features are intentionally excluded from version 1.0 to preserve simplicity and neutrality.

---

## 12. Acknowledgements

This proposal aligns with the mission of Hashgraph Online to promote open, composable, and interoperable standards for the Hedera ecosystem.

---

### Status: **Proposed**