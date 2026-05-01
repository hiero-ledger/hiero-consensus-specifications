---
title: "HCS-XX — Encrypted Persistent AI Context"
description: "A standard for user-owned, encrypted AI context stored on Hedera, enabling persistent AI relationships across sessions, models, and platforms without delegating custody to any third party."
sidebar_position: 999
---

# HCS-XX Standard: Encrypted Persistent AI Context

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
- [Test Vectors](#test-vectors)
- [Conformance](#conformance)
- [References](#references)
- [Governance Record](#governance-record-fill-at-publication)
- [License](#license)

## Authors

- Luther Lee Pollok III ([@gamilu](https://github.com/gamilu))

## Abstract

This standard defines a protocol for persistent, encrypted, user-owned AI context on Hedera. An individual's AI conversation history, preferences, and accumulated knowledge are encrypted with a key derived from an HTS token they own and a wallet signature they produce. The encrypted context is stored via HCS-1. No server ever holds plaintext. The key is never stored — it is derived fresh each session. Context is portable across AI providers and survives any single platform shutdown or terms-of-service change.

## Motivation

Every major AI platform today maintains custody of user context. A user's conversation history, preferences, and accumulated AI relationship live on the platform's servers. This creates five structural problems:

1. **Readability** — The platform can read, analyze, and train on user context at any time.
2. **Mutability** — The platform can modify or delete user context without notice.
3. **Lock-in** — Switching AI providers requires abandoning accumulated context.
4. **Fragility** — Platform shutdown, acquisition, or terms-of-service changes terminate the user's AI relationship.
5. **Compellability** — Governments can compel platforms to produce user context.

Existing encrypted storage solutions typically store encryption keys server-side or delegate key management to a trusted third party. Server-side key storage negates the privacy benefit. Trusted third parties introduce custody risk.

This standard solves these problems using Hedera's existing infrastructure: HTS tokens as ownership proof, HCS-1 as encrypted storage, and HCS topics as the audit trail.

### Relationship to Existing Standards

This standard builds on:
- **HCS-1** — used as the encrypted storage backend
- **HTS** — used as the ownership layer (token = decryption rights)
- **HCS-11** — AI agent profiles may reference a vault conforming to this standard
- **HCS-14** — AI agent communication topics may use this standard for encrypted context persistence

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Overview

The protocol has three layers:

1. **Ownership Layer** — An HTS NFT whose `token_id` is the stable identifier used in key derivation.
2. **Cryptographic Layer** — HKDF-SHA256 key derivation from `token_id` and `wallet_signature`; AES-256-GCM encryption.
3. **Storage Layer** — Encrypted context stored via HCS-1; audit entries logged to an HCS topic.

### 3.1 Ownership Layer

Each user MUST hold an HTS NFT (non-fungible token). The token serves as proof of ownership and as the seed for key derivation.

- The `token_id` (e.g., `0.0.12345`) MUST be used as the stable identifier for the vault.
- Transfer of the token transfers the ability to derive the decryption key and therefore ownership of the vault.
- The token MUST be a non-fungible token (serial number = 1 per user).
- Treasury SHOULD be a separate operator account; the user wallet holds the token.

### 3.2 Key Derivation

The encryption key `K` for a session is derived as follows:

```
input_material = token_id || wallet_signature
K = HKDF-SHA256(IKM=input_material, salt=None, info="anamnesis-v1", length=32)
```

Where:
- `token_id` — the HTS token identifier, UTF-8 encoded (e.g., `"0.0.12345"`)
- `wallet_signature` — the user's Ed25519 signature over a server-issued challenge (bytes)
- `||` — concatenation
- `HKDF-SHA256` — HMAC-based Key Derivation Function per RFC 5869 using SHA-256
- `length=32` — produces a 256-bit key

**Required properties:**
- The key MUST be deterministic: the same token and signature MUST always produce the same key.
- The key MUST NOT be stored on any server or in any persistent medium.
- A unique challenge MUST be issued per session. Because different challenges produce different signatures, each session produces a different key, providing session isolation.
- Without both the `token_id` and the corresponding wallet private key to sign the challenge, the key CANNOT be derived.

### 3.3 Encryption

Context MUST be encrypted using AES-256-GCM:

```
nonce = random_bytes(12)
ciphertext, tag = AES-256-GCM-Encrypt(key=K, plaintext=context, nonce=nonce, aad=token_id)
stored = nonce || tag || ciphertext
```

Where:
- `nonce` — 96-bit random nonce, MUST be unique per encryption operation
- `aad` — Additional Authenticated Data; the `token_id` MUST be used as AAD to bind the ciphertext to its owner
- `tag` — 128-bit authentication tag
- `stored` — the byte sequence written to HCS-1

**Required properties:**
- Authenticated encryption MUST be used. Any tampering with the ciphertext MUST be detectable on decryption.
- Using `token_id` as AAD ensures a ciphertext encrypted for one token CANNOT be decrypted as if it belonged to a different token.

### 3.4 Decryption

```
nonce = stored[:12]
tag = stored[12:28]
ciphertext = stored[28:]
context = AES-256-GCM-Decrypt(key=K, ciphertext=ciphertext, nonce=nonce, tag=tag, aad=token_id)
```

Decryption MUST fail with an authentication error if the ciphertext has been tampered with or the wrong key is used.

### 3.5 Storage Layer (HCS-1)

Encrypted context MUST be stored using HCS-1. HCS-1 is RECOMMENDED over HFS because:
- HCS messages cost ~$0.0001 each, suitable for per-session updates
- HCS-1 supports chunking for large context windows
- Returns a stable Topic ID used as the vault reference

```javascript
import { inscribe, retrieveInscription } from '@hashgraphonline/standards-sdk';

// Store encrypted context
const result = await inscribe(
  { type: 'buffer', content: encryptedContext, fileName: 'context.enc' },
  { accountId: operatorId, privateKey: operatorKey, network: 'mainnet' },
  { mode: 'file', waitForConfirmation: true }
);
const topicId = result.inscription.topic_id;

// Retrieve encrypted context
const inscription = await retrieveInscription(transactionId);
const encryptedContext = inscription.content;
```

The `topicId` returned by HCS-1 MUST be stored as the vault reference for subsequent retrieval.

### 3.6 Audit Trail

Every context update SHOULD be logged to an HCS topic as an append-only audit trail.

Each audit entry MUST conform to:

```json
{
  "standard": "hcs-XX",
  "token_id": "<HTS token identifier>",
  "timestamp": "<consensus timestamp>",
  "content_hash": "<SHA-256 hex of encrypted context>",
  "operation": "create | update | delete",
  "sequence": "<monotonically increasing integer>"
}
```

The audit trail enables:
- Cryptographic proof that context has not been modified without the user's knowledge
- Recovery of any historical version of the context
- Verification of the full update history

### 3.7 Session Protocol

```
Client                          Server                    Hedera
  |                               |                           |
  |--- GET /session/challenge ---->|                           |
  |<-- {challenge, session_id} ----|                           |
  |                               |                           |
  | [sign challenge with wallet]  |                           |
  |                               |                           |
  |--- POST /session/init -------->|                           |
  |    {token_id, signature}      |                           |
  |                               |--- verify token owner --->|
  |                               |<-- {owner: wallet_addr} --|
  |                               |                           |
  |                               | [verify signature matches |
  |                               |  wallet_addr]             |
  |                               |                           |
  |<-- {session_token} ------------|                           |
  |                               |                           |
  | [derive K = HKDF(token_id     |                           |
  |   || signature)]              |                           |
  |                               |                           |
  |--- GET /vault/context -------->|                           |
  |<-- {encrypted_context} --------|                           |
  |                               |                           |
  | [decrypt context with K]      |                           |
  | [inject into AI provider]     |                           |
  | [re-encrypt updated context]  |                           |
  |                               |                           |
  |--- PUT /vault/context -------->|                           |
  |    {encrypted_context}        |--- write via HCS-1 ------>|
  |                               |--- log to HCS topic ----->|
  |<-- {topic_id, content_hash} --|                           |
  |                               |                           |
  | [discard K]                   |                           |
```

**Required session behaviors:**
- The server MUST issue a unique random challenge per session. Challenges MUST NOT be reused.
- The server MUST verify that `wallet_signature` was produced by the wallet currently holding the HTS token before issuing a session token.
- The server MUST NOT store `K` or `wallet_signature` beyond the duration of the session.
- `K` MUST be discarded at session end and MUST NOT be written to any persistent medium.

### 3.8 Context Format

Context is a JSON document. Conforming implementations MUST support the following base schema and MAY extend it:

```json
{
  "version": "1.0",
  "standard": "hcs-XX",
  "token_id": "<HTS token identifier>",
  "created_at": "<ISO 8601 timestamp>",
  "updated_at": "<ISO 8601 timestamp>",
  "messages": [
    {
      "role": "user | assistant | system",
      "content": "<message content>",
      "timestamp": "<ISO 8601 timestamp>",
      "provider": "<ai provider identifier>"
    }
  ],
  "knowledge": [
    {
      "id": "<uuid>",
      "content": "<knowledge chunk>",
      "source": "<source identifier>",
      "created_at": "<ISO 8601 timestamp>"
    }
  ],
  "preferences": {
    "default_provider": "<provider identifier>",
    "custom_fields": {}
  }
}
```

### 3.9 AI Provider Injection

The decrypted context MUST be injected into AI provider calls as a system prompt prefix. The provider MUST NOT retain the context beyond the duration of one API call.

```python
response = ai_provider.chat(
    model=user_preferred_model,
    messages=[
        {"role": "system", "content": f"The following is your context with this user:\n\n{decrypted_context}\n\nContinue from this context."},
        {"role": "user", "content": user_message}
    ]
)
```

Any AI provider with a chat completion API is compatible with this standard.

## Rationale

### Why HKDF-SHA256?

HKDF (RFC 5869) is the standard for key derivation from non-uniform input material. The combination of `token_id` and `wallet_signature` is not uniformly random — HKDF extracts and expands it into a uniformly distributed key. AES-256-GCM is NIST-approved authenticated encryption. These are the most widely implemented and audited primitives available.

### Why token ownership rather than a password?

Passwords can be subpoenaed, compelled, or forgotten. A wallet private key, if managed correctly (hardware wallet, no custodian), cannot be produced under compulsion without the user's cooperation. The token-based model also enables inheritance and transfer — the vault owner can transfer their vault by transferring the token.

### Why HCS-1 rather than HFS?

HFS is designed for static files. AI context is updated every session. HCS-1 costs orders of magnitude less for frequent updates (~$0.0001 per message vs HFS append costs) and is designed specifically for content that changes over time.

### Why session isolation?

A unique challenge per session produces a unique signature, and therefore a unique derived key. Compromise of one session's key material does not affect any other session. This is analogous to the forward secrecy property in TLS.

## Backwards Compatibility

This is a new standard. There are no backwards compatibility constraints.

Implementations migrating from Hedera File Service (HFS) storage to HCS-1 SHOULD re-encrypt and re-store existing context using the HCS-1 backend. The cryptographic layer is unchanged.

## Security Considerations

### Key Exposure

The derived key `K` MUST be treated as a secret of the highest sensitivity. It MUST NOT be logged, transmitted over the network, or written to disk. If `K` is exposed, all context encrypted with it is exposed. Key rotation (deriving a new key by re-encrypting context with a new session) is RECOMMENDED if exposure is suspected.

### Challenge Replay

The server MUST enforce single-use challenges. A replayed signature from a prior session MUST NOT be accepted. Challenges SHOULD expire after 60 seconds if unused.

### Token Theft

If the HTS token is stolen (transferred without authorization), the attacker can derive the key and decrypt the vault. Users SHOULD use hardware wallets or multi-signature accounts for high-sensitivity vaults.

### Ciphertext Transplant

Using `token_id` as AAD in AES-GCM prevents an attacker from taking a ciphertext encrypted for one token and presenting it as belonging to another token. Decryption will fail with an authentication error.

### Dead Man's Switch (Disclosure Protection)

For users whose context may be subject to suppression, the token transfer model supports a dead man's switch pattern: a smart contract that automatically transfers the HTS token to a pre-designated address if the owner fails to check in within a defined window. A reference implementation is available in the Anamnesis Protocol repository. See section 11 of the protocol specification at https://github.com/anamnesis-protocol/anamnesis/blob/main/WHITEPAPER.md.

## Privacy Considerations

### Zero Server-Side Plaintext

The server MUST handle only ciphertext. Plaintext context MUST be produced and consumed exclusively on the client. A conforming server implementation has no technical ability to read user context even under compulsion — there is no plaintext to produce.

### Government Compellability

Because the server holds only ciphertext and never holds the key, a legally valid subpoena or court order served on a conforming server operator produces only ciphertext. The key exists only in the user's wallet — it cannot be compelled from the server operator.

### Context Retention

The AI provider receiving the decrypted context for injection MUST NOT be instructed to retain it. The context passes through the provider's API for one session and is not stored by the conforming implementation.

## Test Vectors

The following test vectors allow implementations to verify correct HKDF-SHA256 key derivation.

**Input:**
```
token_id (UTF-8): "0.0.12345"
wallet_signature (hex): 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20
                        2122232425262728292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f40
info (UTF-8): "anamnesis-v1"
length: 32
```

**Expected output (hex):**
```
K = HKDF-SHA256(IKM="0.0.12345" || wallet_signature_bytes, salt=None, info="anamnesis-v1", length=32)
```

Reference implementation for test vector generation:

```python
import hashlib
import hmac
from cryptography.hazmat.primitives.kdf.hkdf import HKDF
from cryptography.hazmat.primitives import hashes

token_id = b"0.0.12345"
wallet_signature = bytes(range(1, 65))  # 64-byte test signature
ikm = token_id + wallet_signature

hkdf = HKDF(
    algorithm=hashes.SHA256(),
    length=32,
    salt=None,
    info=b"anamnesis-v1",
)
K = hkdf.derive(ikm)
print(K.hex())
```

## Conformance

A conforming implementation of this standard:

1. MUST derive the encryption key using HKDF-SHA256 as specified in section 3.2.
2. MUST encrypt context using AES-256-GCM with `token_id` as AAD as specified in section 3.3.
3. MUST store encrypted context using HCS-1.
4. MUST issue unique single-use challenges per session.
5. MUST verify wallet ownership of the HTS token before issuing a session token.
6. MUST NOT store the derived key `K` in any persistent medium.
7. MUST NOT handle plaintext context server-side.
8. SHOULD log audit entries to an HCS topic as specified in section 3.6.
9. MAY extend the context schema defined in section 3.8.
10. MAY support alternative storage backends provided they satisfy the requirements in section 3.5.

## References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for use in RFCs to Indicate Requirement Levels
- [RFC 5869](https://www.rfc-editor.org/rfc/rfc5869) — HMAC-based Extract-and-Expand Key Derivation Function (HKDF)
- [NIST SP 800-38D](https://csrc.nist.gov/publications/detail/sp/800-38d/final) — Recommendation for GCM and GMAC
- [HCS-1](https://hiero-ledger.github.io/hiero-consensus-specifications/docs/standards/hcs-1) — File data storage via HCS
- [HCS-11](https://hiero-ledger.github.io/hiero-consensus-specifications/docs/standards/hcs-11) — Profile Standard
- [Anamnesis Protocol Reference Implementation](https://github.com/anamnesis-protocol/anamnesis) — Apache 2.0
- [Hashgraph Online Standards SDK](https://hol.org/docs/libraries/standards-sdk/inscribe/) — HCS-1 implementation

## Governance Record (fill at publication)

- Poll topic: hcs://8/<topicId> (or Mirror Node link)
- Outcome: PASS | FAIL on YYYY-MM-DD (UTC)
- Reference: <txn id or final tally link>

## License

This document is licensed under Apache-2.0.
