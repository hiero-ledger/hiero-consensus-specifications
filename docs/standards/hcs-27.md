---
title: "HCS-27 — Transparency / Checkpoint Log"
description: "Standard for transparency logs with Merkle trees and HCS checkpoint streams, enabling verifiable inclusion proofs and on-chain checkpoint verification."
sidebar_position: 27
---

# HCS-27 Standard: Transparency / Checkpoint Log

### Status: Draft

### Version: 1.0

### Table of Contents

- [Authors](#authors)
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Specification](#specification)
  - [Transparency Log Model](#transparency-log-model)
  - [Topic Memo Format](#topic-memo-format)
  - [Checkpoint Message Format](#checkpoint-message-format)
  - [Merkle Proof Structure](#merkle-proof-structure)
  - [Verification Algorithms](#verification-algorithms)
  - [Reading Checkpoint Messages (Level 2c)](#reading-checkpoint-messages-level-2c)
- [Rationale](#rationale)
- [Backwards Compatibility](#backwards-compatibility)
- [Security Considerations](#security-considerations)
- [Privacy Considerations](#privacy-considerations)
- [Test Vectors](#test-vectors)
- [Conformance](#conformance)
- [References](#references)
- [Governance Record (fill at publication)](#governance-record-fill-at-publication)
- [License](#license)

## Authors

- Adethya Srinivasan

## Abstract

HCS-27 defines a transparency log model based on an append-only Merkle tree, a signed tree head (STH) commitment, and checkpoint messages published to a Hedera Consensus Service (HCS) topic. It specifies the topic memo format for checkpoint topics, the checkpoint message schema, the structure of Merkle inclusion proofs, and the verification algorithms (STH verification and VerifyInclusion). Consumers such as the [HCS-14 ANS DNS/Web profile](/docs/standards/hcs-14/profiles/ans-dns-web) can implement Level 2b (Merkle proof verification) and Level 2c (HCS checkpoint verification) against this standard without reverse-engineering.

## Motivation

Transparency logs provide cryptographic evidence that a given entry was included in a log at a specific state. To interoperate across systems (e.g., ANS badge verification, other registries), the log model, proof format, and checkpoint publication mechanism must be standardized. HCS-27 provides that specification so log operators can publish checkpoints to Hedera and verifiers can validate inclusion proofs and optional on-chain checkpoint consistency.

## Specification

### Transparency Log Model

- **Append-only log**: An ordered sequence of log entries. Each entry is assigned a 0-based leaf index and has a single leaf hash. Entries are immutable; new entries append to the log.

- **Merkle tree**: A binary hash tree over the leaves (indices 0 through treeSize − 1). The tree uses the Merkle Tree Hash (MTH) construction aligned with [RFC 6962](https://www.rfc-editor.org/rfc/rfc6962) (Certificate Transparency):
  - **Leaf hash**: `H(0x00 || leaf_input)` where `leaf_input` is the log entry payload (or its canonical encoding) and `H` is the specified hash function (see [Merkle Proof Structure](#merkle-proof-structure)).
  - **Internal node**: `H(0x01 || left_hash || right_hash)` where hashes are concatenated in fixed order (left then right).
  - Tree size and leaf indexing are 0-based and consistent with RFC 6962.

- **Signed tree head (STH)**: A commitment to a specific Merkle root hash and tree size, signed by the log operator. Verifiers use the STH to trust that a given `rootHash` and `treeSize` are the log’s attested state.

- **Checkpoint**: An STH (or equivalent signed payload) published as an HCS message to a Hedera topic. This allows anyone to verify that a given `rootHash` and `treeSize` correspond to the log’s published state (e.g., Level 2c in the ANS profile).

### Topic Memo Format

Checkpoint topics use a colon-delimited memo consistent with [HCS-4](/docs/standards/hcs-4) conventions:

```
hcs-27:<indexed>:<ttl>
```

- **hcs-27**: Protocol identifier for this standard.
- **indexed**: `0` — all messages on the topic MUST be processed (checkpoints are append-only; consumers need the full stream for a given log).
- **ttl**: Cache TTL in seconds (e.g. `86400`). Use `0` if not applicable.

**Example**: `hcs-27:0:86400`

The Hedera topic ID of the checkpoint topic is the value exposed to consumers as `hcs27TopicId` (e.g., in the [HCS-14 ANS badge response](/docs/standards/hcs-14/profiles/ans-dns-web#72-badge-endpoint-response)).

### Checkpoint Message Format

Each HCS message on a checkpoint topic that represents one STH MUST be a JSON object with the following fields:

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `p` | string | Yes | Protocol identifier. MUST be `"hcs-27"`. |
| `op` | string | Yes | Operation. MUST be `"checkpoint"`. |
| `treeSize` | number | Yes | Number of leaves in the tree (non-negative integer). |
| `rootHash` | string | Yes | Merkle root hash, hex-encoded (lowercase, no `0x` prefix). |
| `rootSignature` | string | Yes | Signature over the STH payload (see below), hex-encoded. |
| `timestamp` | string | No | ISO 8601 UTC timestamp for auditing. |
| `logId` | string | No | Opaque log identifier to support multiple logs per topic. |

**STH signed payload**: The value that is signed to produce `rootSignature` MUST be the SHA-256 hash of the concatenation (in order):

1. The literal string `HCS-27-STH` (UTF-8, no trailing null).
2. Big-endian 8-byte representation of `treeSize` (uint64).
3. The raw bytes of `rootHash` (32 bytes for SHA-256; decode from hex).

So:

```
sthPayload = SHA256("HCS-27-STH" || uint64_be(treeSize) || rootHashBytes)
```

Signatures (e.g., ECDSA or Ed25519) are over `sthPayload`; the encoding of the signature itself (e.g., raw R||S or compact form) is log-specific. Verifiers MUST obtain the log’s public key via log metadata, the first message on the topic, or out-of-band and verify `rootSignature` against the recomputed `sthPayload`.

### Merkle Proof Structure

The **merkleProof** object is the inclusion proof supplied to verifiers (e.g., in an ANS badge response). It MUST contain the following fields:

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `leafHash` | string | Yes | Hash of the leaf (as produced by the log’s leaf hash algorithm), hex-encoded. |
| `leafIndex` | number | Yes | 0-based index of the leaf in the tree. |
| `treeSize` | number | Yes | Number of leaves when this proof was generated. |
| `rootHash` | string | Yes | Merkle root that this proof verifies against, hex-encoded. |
| `rootSignature` | string | Yes | Signed tree head signature (same format as in checkpoint message). |
| `path` | array of string | Yes | Audit path: array of sibling hashes from leaf toward root. Each element is hex-encoded. Order: `path[0]` is the sibling of the leaf, `path[1]` is the sibling at the next level, etc. |

**Hash algorithm**: SHA-256. Leaf and internal node hashes follow the RFC 6962 MTH rules above (prefix `0x00` for leaves, `0x01` for internal nodes).

**Audit path order**: `path` is ordered from leaf toward root. At each level, the verifier uses the leaf index to determine whether the current node is the left or right child: for level `i` (0 = leaf level), use `(leafIndex >> i) & 1` — if 0, the current node is the left child and the sibling is `path[i]` on the right; if 1, the current node is the right child and the sibling is `path[i]` on the left. Combine with the same MTH internal node rule: `parent = H(0x01 || left_hash || right_hash)` with left/right order as determined.

### Verification Algorithms

#### STH verification

- **Inputs**: `rootHash` (hex string), `treeSize` (number), `rootSignature` (hex string), and the log’s public key (or key identifier).
- **Process**:
  1. Decode `rootHash` to 32 bytes; encode `treeSize` as big-endian uint64.
  2. Compute `sthPayload = SHA256("HCS-27-STH" || uint64_be(treeSize) || rootHashBytes)`.
  3. Verify `rootSignature` against `sthPayload` using the log’s public key.
- **Output**: Accept if the signature is valid; otherwise reject.

The log’s public key MUST be obtained from log metadata, the first checkpoint message on the topic, or an out-of-band trusted source.

#### VerifyInclusion

- **Inputs**: `leafHash` (hex), `leafIndex` (number), `path` (array of hex strings), `treeSize` (number), `rootHash` (hex).
- **Process**:
  1. If `leafIndex < 0` or `leafIndex >= treeSize`, return reject.
  2. Set `currentHash = leafHash` (decoded to bytes).
  3. For `i = 0` to `len(path) - 1`: let `sibling = path[i]` (decoded). If `(leafIndex >> i) & 1 == 0`, set `currentHash = H(0x01 || currentHash || sibling)`; else set `currentHash = H(0x01 || sibling || currentHash)`.
  4. Compare `currentHash` (hex-encoded) to `rootHash`. Return accept only if equal.
- **Output**: Accept or reject.

Verifiers (e.g., ANS Level 2b) MUST verify the STH first (so the root is trusted), then run VerifyInclusion with the fields from `merkleProof`.

#### Level 2c (checkpoint verification)

- **Inputs**: `rootHash`, `treeSize`, `hcs27TopicId` (Hedera topic ID).
- **Process**: Read HCS messages from the topic identified by `hcs27TopicId`. Parse each message as JSON; ignore messages where `p !== "hcs-27"` or `op !== "checkpoint"`. Find at least one checkpoint message whose `rootHash` and `treeSize` match the supplied values. Optionally verify that checkpoint’s `rootSignature` (STH verification).
- **Output**: Level 2c is satisfied if a matching checkpoint exists (and optionally its signature is valid). If `hcs27TopicId` is missing or invalid, or no matching checkpoint is found, Level 2c is unavailable; this MUST NOT affect Level 2a or Level 2b (see [HCS-14 ANS profile](/docs/standards/hcs-14/profiles/ans-dns-web)).

### Reading Checkpoint Messages (Level 2c)

Consumers that implement Level 2c MUST:

1. Obtain the checkpoint topic ID from the context (e.g., `hcs27TopicId` from the badge response).
2. Query the Hedera network (e.g., Mirror Node REST API or similar) for all messages on that topic, or subscribe to new messages. Messages are ordered by consensus order.
3. Parse each message payload as JSON. Messages that do not contain `p: "hcs-27"` and `op: "checkpoint"` MUST be ignored for the purpose of checkpoint verification.
4. For each checkpoint message, compare `rootHash` and `treeSize` to the values from the badge (or other proof source). If a match is found, Level 2c is satisfied. Optionally verify `rootSignature` for that checkpoint.

The exact Hedera API (Mirror Node, SDK, etc.) is implementation-defined; this standard only defines the message format and the matching semantics.

## Rationale

Merkle trees and signed tree heads provide the same auditability guarantees as Certificate Transparency (RFC 6962), while keeping the spec implementable and testable. Publishing checkpoints to HCS gives a single, consensus-backed source of truth on Hedera so that verifiers can confirm that a given root and tree size were actually published by the log operator (Level 2c). The topic memo format follows HCS-4 so that checkpoint topics are self-describing and discoverable.

## Backwards Compatibility

This is the first version of HCS-27. Future extensions (e.g., additional hash algorithms or optional fields) SHOULD use new fields or type enums and MUST NOT change the meaning of existing fields so that existing proofs and checkpoints remain valid.

## Security Considerations

- **STH key management**: The security of STH verification depends on the log operator’s private key. Compromise allows forging STHs and thus undermining trust in the root. Keys SHOULD be protected and rotated under a documented policy.
- **Inclusion proofs**: Inclusion proofs only prove that a leaf is in a tree with a given root. They do not prove that the root is the “current” or “correct” log state; that is established by STH verification and optionally by Level 2c (checkpoint on Hedera).
- **Replay**: Checkpoint messages can be replayed. Including `timestamp` and/or `logId` helps auditors detect ordering and cross-log reuse; verifiers MAY reject checkpoints that are too old if they have a policy for that.

## Privacy Considerations

Checkpoint messages contain only `rootHash`, `treeSize`, optional `timestamp`, and optional `logId` — no per-entry data. Log entries themselves may contain identifiers (e.g., in ANS payloads); privacy of log entry content is out of scope of HCS-27 and is the responsibility of the log operator and the profile that defines the payload (e.g., HCS-14 ANS).

## Test Vectors

### Small tree (3 leaves)

Assume leaf inputs (before 0x00 prefix) are the strings `"entry0"`, `"entry1"`, `"entry2"`. Hash algorithm: SHA-256 with RFC 6962 MTH.

- `leafHash` for leaf 0: `SHA256(0x00 || "entry0")` → hex value L0  
- `leafHash` for leaf 1: `SHA256(0x00 || "entry1")` → hex value L1  
- `leafHash` for leaf 2: `SHA256(0x00 || "entry2")` → hex value L2  

Internal node N01 = `SHA256(0x01 || L0 || L1)`. For a tree of size 3, the structure is one internal node (N01) and L2 as the other child of the root; root R = `SHA256(0x01 || N01 || L2)`.

**Inclusion proof for leaf 0**:

- `leafHash`: L0 (hex)
- `leafIndex`: 0
- `treeSize`: 3
- `rootHash`: R (hex)
- `path`: [L1, L2] — L1 is sibling of leaf 0, then (N01 combined with L2) gives root; so at level 0 sibling is L1, at level 1 sibling is L2.

Verifier: start with L0; combine with path[0]=L1 → N01 (left child, so H(0x01||L0||L1)); combine N01 with path[1]=L2 → R (left child, so H(0x01||N01||L2)). Compare to rootHash; accept if equal.

*(Implementations should substitute concrete hex values for L0, L1, L2, N01, R when generating deterministic test vectors.)*

### Checkpoint message example

```json
{
  "p": "hcs-27",
  "op": "checkpoint",
  "treeSize": 3,
  "rootHash": "<64 hex chars>",
  "rootSignature": "<hex-encoded signature of sthPayload>",
  "timestamp": "2025-03-02T12:00:00Z"
}
```

STH verification: compute `sthPayload = SHA256("HCS-27-STH" || uint64_be(3) || rootHashBytes)` and verify `rootSignature` against it using the log’s public key.

## Conformance

Implementations that produce or consume HCS-27 checkpoints, or that verify inclusion proofs or STHs, MUST:

- Use the topic memo format `hcs-27:0:<ttl>` for checkpoint topics.
- Use the checkpoint message format with required fields `p`, `op`, `treeSize`, `rootHash`, `rootSignature`.
- Use the STH signed payload construction and verification steps defined in this specification.
- Use the merkleProof structure with required fields `leafHash`, `leafIndex`, `treeSize`, `rootHash`, `rootSignature`, `path`.
- Implement VerifyInclusion and STH verification according to the algorithms above.
- Use SHA-256 and the RFC 6962 MTH leaf/internal node rules for hashing.

## References

- [RFC 6962](https://www.rfc-editor.org/rfc/rfc6962) — Certificate Transparency (Merkle tree and audit path).
- [HCS-4](/docs/standards/hcs-4) — Topic memos and operations conventions.
- [HCS-14 Profile: ANS DNS/Web](/docs/standards/hcs-14/profiles/ans-dns-web) — Normative consumer of HCS-27 for Level 2b/2c.

## Governance Record (fill at publication)

- Poll topic: hcs://8/<topicId> (or Mirror Node link)
- Outcome: PASS | FAIL on YYYY-MM-DD (UTC)
- Reference: <txn id or final tally link>

## License

This document is licensed under Apache-2.0.
