# HCS-28 (Adapter): `verification.publisher-bound`

## Purpose

Reward verified publisher identity binding for the skill version.

## Contribution

- Adapter id: `verification.publisher-bound`
- Contribution mode: `conditional`
- Default weight: `0.20`
- Output key: `verification.publisher-bound.score`

## Applicability

Applies to all HCS-26 skill subjects.

## Inputs

- `signals.publisherBound.ok` boolean.

## Normalization

- `score = 100` when `signals.publisherBound.ok=true`
- `score = 0` otherwise

Implementations MUST emit `verification.publisher-bound.score` as a finite value in `[0,100]`.

## Semantics

`signals.publisherBound.ok` MUST be `true` only when a verifier has established a verifiable binding between:

- the skill subject `(network, discovery_topic_id, skill_uid, version)`, and
- the publisher identity recorded for the skill in HCS-26 discovery data (for example, `account_id`).

At minimum, a conforming verifier MUST authenticate control of the publisher identity at verification time (for example, by a cryptographic signature) and MUST reject unverifiable or mismatched publisher assertions.

In baseline interoperability mode, verifiers SHOULD use a signature challenge that binds the publisher to the subject tuple. For an HCS-26 `account_id` publisher identity, a verifier SHOULD:

1. resolve the publisher account public key from the network at verification time;
2. construct the following UTF-8 message to sign:

   ```
   hcs-28:publisher-bound:<network>:<discovery_topic_id>:<skill_uid>:<version>:<issued_at>:<expires_at>
   ```

3. verify the signature with the resolved public key; and
4. require `expires_at` to be in the future.

If the account public key cannot be resolved, if the signature is invalid, or if the subject tuple does not match exactly, the verifier MUST treat `signals.publisherBound.ok` as `false`.

## Evidence (Recommended)

Implementations SHOULD preserve enough evidence to audit the binding decision, for example:

```json
{
  "account_id": "0.0.78910",
  "checkedAt": "2026-03-02T12:00:00.000Z",
  "expiresAt": "2026-03-09T12:00:00.000Z",
  "signatureKind": "ed25519"
}
```
