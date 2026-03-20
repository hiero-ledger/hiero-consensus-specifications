---
title: "HCS-27 — Merkle Tree Profile and Proofs"
description: Deterministic Merkle tree profile, proof verification algorithms, and test vectors for HCS-27 transparency logs.
sidebar_position: 2
---

# HCS-27: Merkle Tree Profile and Proofs (Normative)

This document defines the deterministic Merkle tree profile ("HCS-27 Merkle v1"), proof object shapes, signed tree head format, and verification algorithms used by [HCS-27](./index.md).

This document is concerned exclusively with cryptographic tree construction and proof verification. It does not define log entry schemas, registry semantics, API response formats, or application-level interpretation of log contents.

## Merkle Tree Profile (Deterministic)

All HCS-27 logs MUST use the following deterministic Merkle tree profile ("HCS-27 Merkle v1").

This Merkle hashing construction is equivalent to the Merkle Tree Hash defined in RFC 9162 §2 (Certificate Transparency v2), using SHA-256 with `0x00` leaf prefix and `0x01` node prefix. The hashing math is identical to the earlier RFC 6962; the distinction matters only for tile-based proof formats, which are out of scope for this profile.

### Hash Function

The hash function MUST be SHA-256.

### Encoding Conventions

#### Hash Encoding

| Context | Encoding |
| :---- | :---- |
| `leafHash` in proof objects | Lowercase hex (no `0x` prefix) |
| `path` entries in proof objects | Standard base64 (RFC 4648 §4, with `=` padding) |
| `rootHash` in proof objects | Standard base64 (RFC 4648 §4, with `=` padding) |
| `rootHashB64u` in HCS-27 on-ledger checkpoints | Base64url (RFC 4648 §5, no padding) |

The underlying hash bytes are identical across encodings. Implementations bridging between on-ledger checkpoints and off-ledger proofs MUST convert correctly between base64url and standard base64.

**Open question:** The current implementation uses standard base64 for off-ledger proof objects while HCS-27 on-ledger checkpoints use base64url. Alignment of encoding conventions should be resolved before finalization.

#### Integer Encoding

Integer fields in proof objects (`treeSize`, `leafIndex`, `oldTreeSize`, `newTreeSize`) MUST be encoded as base-10 strings representing non-negative integers.

Verifiers MUST reject values that are not canonical decimal strings (`"0"` or no leading zeros).

### Entry Canonicalization

Each log entry MUST be a JSON object. Before hashing, entries MUST be canonicalized per the JSON Canonicalization Scheme (JCS) as defined by RFC 8785.

HCS-27 does not define the entry schema. The entry structure is determined by the registry that produces log entries. Canonicalization MUST include all fields present in the entry object, regardless of schema.

### Leaf Hashing

Let `E` be the canonical UTF-8 bytes of an entry object (after JCS canonicalization). The JCS output MUST be encoded as UTF-8 bytes with no BOM before hashing.

The leaf hash MUST be computed as:

```
LeafHash = SHA256( 0x00 || E )
```

Where `0x00` is a single zero byte and `||` denotes byte concatenation.

### Node Hashing

Let `L` and `R` be the left and right child hashes (raw bytes).

The parent node hash MUST be computed as:

```
NodeHash = SHA256( 0x01 || L || R )
```

Where `0x01` is a single byte with value 1.

### Root Hash

For a tree of size `N` entries indexed `0..N-1`, the Merkle root MUST be computed using the left-balanced split at the largest power of two strictly less than `N`.

Define:

- `H_leaf(E) = SHA256(0x00 || E)` where `E` is canonical entry bytes.
- `H_node(L, R) = SHA256(0x01 || L || R)` where `L` and `R` are raw hash bytes.

Let `LargestPowerOfTwoLessThan(n)` return the largest power of two strictly less than `n`.

The Merkle root MUST be computed by:

```
function MerkleTreeHash(entries):
  if entries.length == 0:
    return SHA256("")
  if entries.length == 1:
    return H_leaf(entries[0])
  k = LargestPowerOfTwoLessThan(entries.length)
  left = MerkleTreeHash(entries[0:k])
  right = MerkleTreeHash(entries[k:entries.length])
  return H_node(left, right)
```

The empty tree root (size `0`) MUST be:

```
EmptyRoot = SHA256( "" )
```

## Proof Objects (Off-ledger)

This section defines proof object shapes for exchange between log operators, resolvers, and auditors. These are off-ledger data structures and are not published on HCS.

### Inclusion Proof Object (Normative)

An inclusion proof object MUST include:

```json
{
  "leafHash": "a15fbdc27a9568565b37914425e1fe9e3e658df17e7b90edf2caa021ccadaee8",
  "leafIndex": "22",
  "treeSize": "672",
  "path": [
    "B43FC2Z6vztLDeI9hdNiES7bYVp1XIqwpyroDG5gQoM=",
    "mn+1fYn6xZfd7fIpbqYs7QOgt2GQMPg/rnknhrq8kRw="
  ],
  "rootHash": "c6poiSuxRFIAEY5QNXM+igtKov7MjKbNheIx7isLijo=",
  "rootSignature": "eyJhbGciOiJFUzI1NiIs...",
  "treeVersion": 1
}
```

| Field | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| `leafHash` | string | Yes | Lowercase hex leaf hash (`SHA256(0x00 \|\| canonical_entry_bytes)`). |
| `leafIndex` | string | Yes | 0-based position of the entry in the log (base-10 string). |
| `treeSize` | string | Yes | Tree size of the checkpoint this proof is against (base-10 string). |
| `path` | array of strings | Yes | Ordered sibling hash values for the inclusion proof, encoded as standard base64 (RFC 4648 §4, with `=` padding). The array MUST be ordered exactly as consumed by `VerifyInclusion`, starting at the leaf level and ascending toward the root. Empty array for a single-entry tree. |
| `rootHash` | string | Yes | Root hash of the tree at `treeSize` (standard base64). |
| `rootSignature` | string | No | Signed tree head as a compact JWS. See [Signed Tree Head](#signed-tree-head-normative). |
| `treeVersion` | integer | Yes | Merkle tree profile version. MUST be `1` for HCS-27 Merkle v1. |

### Consistency Proof Object (Normative)

A consistency proof object MUST include:

```json
{
  "oldTreeSize": "600",
  "newTreeSize": "672",
  "oldRootHash": "base64...",
  "newRootHash": "c6poiSuxRFIAEY5QNXM+igtKov7MjKbNheIx7isLijo=",
  "consistencyPath": ["base64...", "base64..."],
  "treeVersion": 1
}
```

| Field | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| `oldTreeSize` | string | Yes | Earlier checkpoint tree size (base-10 string). |
| `newTreeSize` | string | Yes | Later checkpoint tree size (base-10 string). |
| `oldRootHash` | string | Yes | Root hash of the earlier checkpoint (standard base64). |
| `newRootHash` | string | Yes | Root hash of the later checkpoint (standard base64). |
| `consistencyPath` | array of strings | Yes | Ordered hash values for the consistency proof, encoded as standard base64 (RFC 4648 §4, with `=` padding). The array MUST be ordered exactly as consumed by `VerifyConsistency`. |
| `treeVersion` | integer | Yes | Merkle tree profile version. MUST be `1`. |

## Proof Verification Algorithms

### Inclusion Proof Verification (Normative)

Given:

- `leafIndex` (base-10 string; 0-based),
- `treeSize` (base-10 string; N),
- a `leafHash` (hex), and
- a `path` array of sibling hashes (base64),

verifiers MUST:

1. Parse `leafIndex` and `treeSize` as non-negative integers (reject if non-canonical).
2. Reject if `leafIndex >= treeSize`.
3. Convert `leafHash` from hex to raw bytes.
4. Verify inclusion using the following procedure:

```
function VerifyInclusion(leaf_index, tree_size, hash, path, expected_root_b64):
  fn = leaf_index
  sn = tree_size - 1
  r = hash

  for each p_b64 in path:
    if sn == 0:
      return false

    p = base64_to_bytes(p_b64)

    if (LSB(fn) == 1) or (fn == sn):
      r = H_node(p, r)
      if (LSB(fn) == 0):
        while (LSB(fn) == 0) and (fn != 0):
          fn = floor(fn / 2)
          sn = floor(sn / 2)
    else:
      r = H_node(r, p)

    fn = floor(fn / 2)
    sn = floor(sn / 2)

  return (sn == 0) and (bytes_to_base64(r) == expected_root_b64)
```

Where `LSB(x)` returns the least significant bit of `x` (0 or 1).

If a verifier has the original entry object rather than a precomputed `leafHash`, it MUST first canonicalize the entry per RFC 8785 and compute `leafHash = hex(SHA256(0x00 || canonical_bytes))`.

### Consistency Proof Verification (Normative)

Given:

- an earlier checkpoint (`oldTreeSize`, `oldRootHash`), and
- a later checkpoint (`newTreeSize`, `newRootHash`), and
- a `consistencyPath` array of hashes (base64),

where `oldTreeSize` and `newTreeSize` are base-10 strings representing non-negative integers.

Verifiers MUST:

1. Parse `oldTreeSize` and `newTreeSize` as non-negative integers from their base-10 string form. Verifiers MUST reject values that are not canonical decimal strings (`"0"` or no leading zeros).
2. Reject if `newTreeSize < oldTreeSize`.

If `oldTreeSize == 0`, any `newRootHash` is consistent with the empty prefix and verifiers MUST accept.

If `oldTreeSize == newTreeSize`, verifiers MUST accept only if `oldRootHash == newRootHash` and `consistencyPath` is empty.

Otherwise, verifiers MUST verify append-only consistency using:

```
function VerifyConsistency(old_size, new_size, old_root_b64, new_root_b64, consistency_path):
  if old_size == 0:
    return true
  if old_size == new_size:
    return (old_root_b64 == new_root_b64) and (consistency_path.length == 0)
  if old_size > new_size:
    return false
  if consistency_path.length == 0:
    return false

  path = consistency_path
  if IsExactPowerOfTwo(old_size):
    path = [old_root_b64] + consistency_path

  fn = old_size - 1
  sn = new_size - 1

  while (LSB(fn) == 1):
    fn = floor(fn / 2)
    sn = floor(sn / 2)

  fr = base64_to_bytes(path[0])
  sr = base64_to_bytes(path[0])
  i = 1

  while i < path.length:
    p = base64_to_bytes(path[i])

    if (sn == 0):
      return false

    if (LSB(fn) == 1) or (fn == sn):
      fr = H_node(p, fr)
      sr = H_node(p, sr)
      if (LSB(fn) == 0):
        while (LSB(fn) == 0) and (fn != 0):
          fn = floor(fn / 2)
          sn = floor(sn / 2)
    else:
      sr = H_node(sr, p)

    fn = floor(fn / 2)
    sn = floor(sn / 2)
    i = i + 1

  return (sn == 0) and (bytes_to_base64(fr) == old_root_b64) and (bytes_to_base64(sr) == new_root_b64)
```

Where `IsExactPowerOfTwo(x)` returns `true` if and only if `x` is a power of two.

## Signed Tree Head (Normative)

The signed tree head (STH) cryptographically binds a checkpoint's root hash and tree size to the log operator's signing key. When present in a proof object as `rootSignature`, it is a compact JWS (JSON Web Signature, RFC 7515).

### STH JWS Header

| Field | Type | Description |
| :---- | :---- | :---- |
| `alg` | string | Signature algorithm (e.g., `ES256`). |
| `kid` | string | Key identifier for the signing key. |
| `raid` | string | Registration Authority log instance identifier. |
| `timestamp` | integer | Unix epoch seconds when the STH was produced. |
| `typ` | string | MUST be `"JWT"`. |

### STH JWS Payload

The STH payload is a standard JWS/JWT payload. Numeric fields in the payload (`treeSize`, `timestamp`) are JSON integers per JWT conventions (RFC 7519 §2), not base-10 strings.

| Field | Type | Description |
| :---- | :---- | :---- |
| `checkpointFormat` | string | Checkpoint format identifier (e.g., `c2sp/v1`). |
| `origin` | string | FQDN of the transparency log service that produced this STH. |
| `rootHash` | string | Root hash (standard base64). MUST match `rootHash` in the enclosing proof object. |
| `timestamp` | integer | Unix epoch seconds of the checkpoint. |
| `treeSize` | integer | Tree size at the checkpoint. Verifiers MUST compare by parsed numeric value against the enclosing proof object's `treeSize` string. |

### STH Verification

Verifiers validating `rootSignature` MUST:

1. Decode the compact JWS and extract the header and payload.
2. Verify that `payload.rootHash` matches `rootHash` in the proof object.
3. Verify that `payload.treeSize` equals the numeric value of `treeSize` in the proof object (parse the base-10 string, then compare as integers).
4. Verify the JWS signature using the key identified by `header.kid`.

`payload.origin` is informational unless the verifier is configured with an expected origin for the stream; in that case the verifier MUST reject if `payload.origin` is present and does not match.

Key discovery mechanisms for resolving `header.kid` to a public key are out of scope for this document.

### Relationship to HCS-27 `metadata.sig`

HCS-27 checkpoint messages include an optional `metadata.sig` object with `alg`, `kid`, and `b64u` fields. When present:

- `metadata.sig.alg` corresponds to the JWS header `alg`.
- `metadata.sig.kid` corresponds to the JWS header `kid`.
- `metadata.sig.b64u` is the base64url-encoded JWS signature bytes (the third segment of the compact JWS).

The full compact JWS (`rootSignature`) contains additional context (header claims, payload) beyond what is stored on-ledger. The on-ledger `metadata.sig` captures only the signature component and key reference for compact storage within the 1 KB HCS message limit.

## Test Vectors

The following test vectors are normative for verifying hashing and proof algorithms.

### TV1: Empty root

```
EmptyRoot = SHA256("")
EmptyRoot (hex) = e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
EmptyRoot (base64) = 47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=
```

### TV2: Leaf hash computation

Given an arbitrary canonical entry (UTF-8 bytes after JCS canonicalization):

```
LeafHash = SHA256(0x00 || canonical_entry_bytes)
```

The leaf hash is encoded as lowercase hex (64 characters).

**Note:** A complete test vector with a concrete entry, its JCS canonical form, and the resulting leaf hash will be provided before finalization. Implementations SHOULD generate test fixtures from a known transparency log to validate their canonicalization and hashing.

### TV3: Inclusion proof verification

Given an inclusion proof object with known values, verifiers MUST reproduce `rootHash` from `leafHash`, `leafIndex`, `treeSize`, and `path` using the `VerifyInclusion` algorithm.

**Note:** A complete worked example with all intermediate hash computations will be provided before finalization.
