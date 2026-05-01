---
title: "HCS-XX — Encrypted User-Owned AI Context"
description: "A standard for persistent, encrypted, user-owned AI context stored on Hedera, enabling continuous AI relationships across sessions and providers without delegating custody to any third party."
sidebar_position: 999
---

# HCS-XX Standard: Encrypted User-Owned AI Context

### Status: Draft

### Version: 1.0.0

### Table of Contents

- [Authors](#authors)
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Specification](#specification)
- [Rationale](#rationale)
- [Backwards Compatibility](#backwards-compatibility)
- [Security Considerations](#security-considerations)
- [Privacy Considerations](#privacy-considerations)
- [Conformance](#conformance)
- [References](#references)
- [Governance Record](#governance-record-fill-at-publication)
- [License](#license)

## Authors

- Luther Lee Pollok III ([@gamilu](https://github.com/gamilu))

## Abstract

This standard defines a protocol for persistent, encrypted, user-owned AI context on Hedera. A user's AI conversation history, preferences, and accumulated knowledge are encrypted using a key derived from a token the user owns. Encrypted context is stored via [HCS-1](./hcs-1.md). No server holds plaintext at any point. The encryption key is derived fresh each session and is never stored. Context is portable across AI providers and survives any single platform shutdown or terms-of-service change.

This standard specifies:
- The HCS-1 vault message format for encrypted context storage
- The HCS audit trail message format for tamper-evident context history
- The session operations and their associated message schemas
- The memo formats for vault and audit topics

Cryptographic primitives (key derivation, encryption algorithm) are specified in the Anamnesis Protocol reference specification.[^1] Token ownership is established via an HTS non-fungible token and is not specified by this standard.

## Motivation

AI platforms today maintain exclusive custody of user context. A user's conversation history, knowledge, and accumulated AI relationship live on the platform's servers. This creates structural problems:

1. **Readability** — The platform can read, analyze, and train on user context without the user's knowledge.
2. **Mutability** — The platform can modify or delete user context without notice.
3. **Lock-in** — Switching AI providers requires abandoning accumulated context.
4. **Fragility** — Platform shutdown, acquisition, or terms-of-service changes terminate the user's AI relationship.
5. **Compellability** — Governments can compel platforms to produce user context.

This standard enables a user to hold their AI context as a cryptographic asset — stored on Hedera, readable only by the token holder, and portable across any provider that implements the standard.

### Relationship to Existing Standards

This standard extends [HCS-2](./hcs-2.md) for topic registry operations and builds on [HCS-1](./hcs-1.md) for encrypted blob storage. Implementations MAY integrate with [HCS-11](./hcs-11.md) agent profiles by referencing a conforming vault in the agent profile's metadata field.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Architecture Overview

A conforming implementation uses two Hedera topic structures:

1. **Vault Topic** — An HCS-1 topic holding the user's encrypted context blob. Updated each session.
2. **Audit Topic** — An HCS-2 indexed topic holding an append-only log of every context operation, with content hashes for tamper detection.

Both topics are associated with a single HTS non-fungible token (the **ownership token**). The token's `t_id` is the stable identifier used to locate and authenticate the vault. Transfer of the token transfers all vault access rights.

```
┌─────────────────────────────────────────────┐
│               Ownership Token (HTS NFT)      │
│               t_id: 0.0.XXXXX               │
└──────────────────┬──────────────────────────┘
                   │ references
        ┌──────────┴──────────┐
        ▼                     ▼
┌───────────────┐    ┌─────────────────┐
│  Vault Topic  │    │   Audit Topic   │
│  (HCS-1)      │    │   (HCS-2)       │
│               │    │                 │
│  Encrypted    │    │  Content hash   │
│  context blob │    │  per operation  │
└───────────────┘    └─────────────────┘
```

### Topic System

#### Topic Types

This standard uses two HCS topic types to manage AI context storage and integrity. Both topic types extend [HCS-2](./hcs-2.md):

| Topic Type | Description | Key Configuration |
|---|---|---|
| **Vault Topic** | Stores the user's current encrypted context blob | Public submit key — client writes each session |
| **Audit Topic** | Append-only log of all vault operations with content hashes | Public submit key; indexed (HCS-2 enum `0`) — all messages required to compute state |

#### Topic Relationships

- The **ownership token** (`t_id`) anchors both topics. Its value is embedded in each topic memo as the stable vault identifier.
- The **vault topic** holds only the current encrypted context blob. Each `update` operation supersedes the prior content.
- The **audit topic** accumulates all operations in order, providing a tamper-evident history. Gaps in `sequence` values indicate a potential integrity violation.
- Transfer of the ownership token transfers all vault access rights. The new holder resolves and decrypts the vault using the same session protocol.

### Topic Memo Formats

#### Vault Topic Memo

The vault topic MUST be created with the following memo format:

```
hcs-XX:vault:{t_id}
```

| Field | Description | Example |
|-------|-------------|---------|
| `hcs-XX` | Protocol identifier for this standard | `hcs-XX` |
| `vault` | Topic type identifier | `vault` |
| `t_id` | HTS token identifier of the ownership token | `0.0.12345` |

Example: `hcs-XX:vault:0.0.12345`

#### Audit Topic Memo

The audit topic MUST be created with the following memo format, extending the [HCS-2](./hcs-2.md) indexed registry memo:

```
hcs-XX:audit:{t_id}:0:86400
```

| Field | Description | Example |
|-------|-------------|---------|
| `hcs-XX` | Protocol identifier | `hcs-XX` |
| `audit` | Topic type identifier | `audit` |
| `t_id` | HTS token identifier of the ownership token | `0.0.12345` |
| `0` | HCS-2 indexed enum — all messages required to compute state | `0` |
| `86400` | TTL in seconds (default: one day) | `86400` |

Example: `hcs-XX:audit:0.0.12345:0:86400`

### Operations

All operations are submitted as HCS messages to the relevant topic. Operations that are not finalized MUST NOT be used in production.

#### Vault Operations (submitted to Vault Topic via HCS-1)

| Operation | Description | Finalized |
|-----------|-------------|-----------|
| `create` | Initializes a new vault with the first encrypted context blob | ✅ |
| `update` | Replaces the vault content with a new encrypted context blob | ✅ |
| `delete` | Marks the vault as deleted; topic remains but context is cleared | ✅ |
| `transfer` | Signals that vault ownership has transferred to a new token holder | ✅ |

#### Audit Operations (submitted to Audit Topic)

| Operation | Description | Finalized |
|-----------|-------------|-----------|
| `log` | Appends a tamper-evident record of a vault operation | ✅ |

### Message Formats

#### Vault Create Message

Submitted to the vault topic on initial vault creation.

```json
{
  "p": "hcs-XX",
  "op": "create",
  "t_id": "<HTS token identifier>",
  "vault_ref": "<HCS-1 topic ID of encrypted context>",
  "content_hash": "<SHA-256 hex of encrypted context blob>",
  "version": "1.0",
  "m": "<optional memo>"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `p` | REQUIRED | Must be `hcs-XX` |
| `op` | REQUIRED | Must be `create` |
| `t_id` | REQUIRED | HTS token identifier; must match vault topic memo |
| `vault_ref` | REQUIRED | HCS-1 topic ID where encrypted context is stored |
| `content_hash` | REQUIRED | SHA-256 hex digest of the encrypted context blob |
| `version` | REQUIRED | Protocol version; must be `1.0` for this version |
| `m` | OPTIONAL | Memo; maximum 500 characters |

#### Vault Update Message

Submitted to the vault topic on each session close.

```json
{
  "p": "hcs-XX",
  "op": "update",
  "t_id": "<HTS token identifier>",
  "vault_ref": "<HCS-1 topic ID of updated encrypted context>",
  "content_hash": "<SHA-256 hex of updated encrypted context blob>",
  "prev_hash": "<SHA-256 hex of previous encrypted context blob>",
  "m": "<optional memo>"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `p` | REQUIRED | Must be `hcs-XX` |
| `op` | REQUIRED | Must be `update` |
| `t_id` | REQUIRED | HTS token identifier |
| `vault_ref` | REQUIRED | HCS-1 topic ID of updated context |
| `content_hash` | REQUIRED | SHA-256 hex of new encrypted context blob |
| `prev_hash` | RECOMMENDED | SHA-256 hex of previous blob; enables chain verification |
| `m` | OPTIONAL | Memo; maximum 500 characters |

#### Vault Delete Message

```json
{
  "p": "hcs-XX",
  "op": "delete",
  "t_id": "<HTS token identifier>",
  "m": "<optional memo>"
}
```

#### Vault Transfer Message

Submitted when the ownership token is transferred to a new holder.

```json
{
  "p": "hcs-XX",
  "op": "transfer",
  "t_id": "<HTS token identifier>",
  "from": "<previous holder wallet address>",
  "to": "<new holder wallet address>",
  "m": "<optional memo>"
}
```

#### Audit Log Message

Submitted to the audit topic after each vault operation.

```json
{
  "p": "hcs-XX",
  "op": "log",
  "t_id": "<HTS token identifier>",
  "vault_op": "<vault operation: create | update | delete | transfer>",
  "content_hash": "<SHA-256 hex of encrypted context blob>",
  "sequence": "<monotonically increasing integer>",
  "m": "<optional memo>"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `p` | REQUIRED | Must be `hcs-XX` |
| `op` | REQUIRED | Must be `log` |
| `t_id` | REQUIRED | HTS token identifier |
| `vault_op` | REQUIRED | The vault operation being logged |
| `content_hash` | REQUIRED | SHA-256 hex of the encrypted context blob after the operation |
| `sequence` | REQUIRED | Monotonically increasing integer; gaps indicate missing entries |
| `m` | OPTIONAL | Memo; maximum 500 characters |

### Vault Reference Format

A vault is referenced using a Hedera Resource Locator (HRL) in the following format:

```
hcs://XX/{t_id}
```

Example: `hcs://XX/0.0.12345`

Implementations resolving a vault reference MUST:
1. Look up the vault topic memo matching `hcs-XX:vault:{t_id}`
2. Retrieve the latest HCS-1 content from the vault topic
3. Verify the content hash against the most recent audit log entry

### Session Protocol

A session consists of the following operations in order:

| Step | Actor | Operation | Description |
|------|-------|-----------|-------------|
| 1 | Client | — | Requests a single-use challenge from the session endpoint |
| 2 | Server | — | Issues a unique random challenge; records session ID |
| 3 | Client | — | Signs challenge with wallet private key |
| 4 | Client | — | Submits `t_id` and wallet signature to session endpoint |
| 5 | Server | — | Verifies wallet is current holder of ownership token on-chain |
| 6 | Server | — | Verifies signature was produced by the verified wallet |
| 7 | Server | — | Issues session token; does not store derived key |
| 8 | Client | — | Derives encryption key from `t_id` and wallet signature |
| 9 | Client | — | Retrieves encrypted context blob from vault topic (HCS-1) |
| 10 | Client | — | Decrypts context; injects into AI provider request |
| 11 | Client | — | Re-encrypts updated context after AI interaction |
| 12 | Client | `update` | Submits updated encrypted blob to vault topic |
| 13 | Client | `log` | Submits audit entry to audit topic |
| 14 | Client | — | Discards encryption key |

Challenges MUST be single-use. A challenge MUST NOT be accepted more than once. The server MUST NOT store the derived encryption key at any step.

### Validation

#### Message Validation

- `p` MUST be the string `hcs-XX` (where XX is the assigned standard number).
- `op` MUST be one of the defined operation values for the target topic type.
- `t_id` MUST match the Hedera token ID format: three groups of integers separated by periods (e.g., `0.0.12345`).
- `content_hash` MUST be a 64-character lowercase hexadecimal string (SHA-256).
- `vault_ref` MUST reference a valid HCS-1 topic ID.
- `m` MUST NOT exceed 500 characters.
- Messages with unknown `op` values MUST be ignored by conforming implementations.

#### Audit Chain Validation

A conforming verifier MUST:
1. Retrieve all `log` messages from the audit topic in sequence order.
2. Verify that `sequence` values are contiguous. Gaps indicate a possible integrity violation.
3. Verify that the `content_hash` in each `log` message matches the `content_hash` in the corresponding vault operation.
4. For `update` operations with `prev_hash`: verify that `prev_hash` matches the `content_hash` of the preceding `log` entry.

## Rationale

### Why extend HCS-2?

HCS-2 provides the registry and indexing semantics that make audit trail retrieval deterministic. An indexed HCS-2 topic (enum 0) requires all messages to be read to compute state — which is exactly the property needed for an append-only, verifiable audit log.

### Why HCS-1 for vault storage?

HCS-1 is designed for content that updates frequently. AI context is re-encrypted and re-stored each session. HCS-1 is more cost-effective than HFS for this access pattern and provides chunking for large context windows.

### Why token-derived key derivation?

The token ownership model means that decryption rights are inseparable from token ownership. There is no separate password to subpoena, compel, or forget. Transfer of the token transfers the vault — enabling inheritance, delegation, and emergency access patterns without requiring any server-side key management.

### Why is key derivation out of scope for this standard?

This standard specifies message formats and operations on Hedera infrastructure. The cryptographic key derivation algorithm is an implementation concern documented in the reference specification.[^1] Separating these concerns allows the HCS message format to remain stable across future cryptographic algorithm updates.

## Backwards Compatibility

This is a new standard with no prior versions. No backwards compatibility constraints apply.

## Security Considerations

### Challenge Replay

The session endpoint MUST enforce single-use challenges. A replayed wallet signature from a prior session MUST be rejected. Implementations SHOULD expire unused challenges after 60 seconds.

### Token Theft

Transfer of the ownership token transfers vault access. Users SHOULD protect their wallet private key using hardware security where high-sensitivity context is stored. Implementations MAY support multi-signature token custody.

### Audit Gap Detection

Missing `sequence` values in the audit topic indicate either dropped messages or deliberate omission. Conforming verifiers MUST flag gaps rather than silently accepting incomplete audit chains.

### Emergency Access (Dead Man's Switch)

For users whose context may be subject to suppression, the token transfer model supports automated transfer via a smart contract that triggers if the owner fails to check in within a defined interval. A reference pattern is documented in the Anamnesis Protocol specification.[^1]

## Privacy Considerations

### Server-Side Plaintext

A conforming server implementation handles only ciphertext. Plaintext context MUST be produced and consumed exclusively on the client. A server operator served with a legal demand for user context has no plaintext to produce.

### AI Provider Exposure

The decrypted context is injected into AI provider requests for one session. The provider receives context for the duration of one API interaction. Conforming implementations MUST NOT instruct the provider to retain context beyond the session.

### Audit Trail Metadata

The audit topic is append-only and publicly readable on Hedera. It contains content hashes and timestamps but not plaintext context. Implementors SHOULD consider whether the pattern of vault updates (frequency, timing) constitutes sensitive metadata for their use case.

## Conformance

A conforming implementation MUST:

1. Create vault topics with the memo format specified in section [Vault Topic Memo].
2. Create audit topics with the memo format specified in section [Audit Topic Memo].
3. Submit vault operation messages conforming to the schemas in section [Message Formats].
4. Submit an audit log entry after every vault operation.
5. Enforce single-use session challenges.
6. Verify that the wallet signature was produced by the current holder of the ownership token before issuing a session token.
7. Discard the encryption key at session end; MUST NOT write it to any persistent medium.
8. Handle only ciphertext server-side; MUST NOT decrypt context on the server.
9. Reject messages with unrecognized `op` values without error.
10. Flag non-contiguous `sequence` values during audit chain verification.

A conforming implementation MAY:

1. Support vault reference resolution as specified in section [Vault Reference Format].
2. Implement emergency transfer patterns as described in the Security Considerations.
3. Integrate vault references into [HCS-11](./hcs-11.md) agent profile metadata.

## References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs
- [HCS-1](./hcs-1.md) — File storage via HCS
- [HCS-2](./hcs-2.md) — Advanced Topic Registries
- [HCS-11](./hcs-11.md) — Profile Standard
- [HCS-13](./hcs-13.md) — Schema Registry and Validation

[^1]: Anamnesis Protocol specification and reference implementation: https://github.com/anamnesis-protocol/anamnesis (Apache 2.0). Specifies HKDF-SHA256 key derivation, AES-256-GCM encryption, session protocol, and emergency access patterns.

## Governance Record (fill at publication)

- Poll topic: hcs://8/<topicId> (or Mirror Node link)
- Outcome: PASS | FAIL on YYYY-MM-DD (UTC)
- Reference: <txn id or final tally link>

## License

This document is licensed under Apache-2.0.
