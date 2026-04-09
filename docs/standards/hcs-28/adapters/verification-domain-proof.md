# HCS-28 (Adapter): `verification.domain-proof`

## Purpose

Reward successful domain-control proof for the skill publisher.

## Contribution

- Adapter id: `verification.domain-proof`
- Contribution mode: `conditional`
- Default weight: `0.10`
- Output key: `verification.domain-proof.score`

## Applicability

Applies when a skill declares a domain/homepage or when a domain proof signal exists.

## Inputs

- `signals.domainProof.ok` boolean.

## Normalization

- `score = 100` when `signals.domainProof.ok=true`
- `score = 0` otherwise

Implementations MUST emit `verification.domain-proof.score` as a finite value in `[0,100]`.

## Semantics

`signals.domainProof.ok` MUST be `true` only when a verifier has established control of a domain associated with the skill (typically derived from `homepage`) using a verifiable challenge-response method such as:

- DNS TXT record challenge, or
- HTTPS well-known file challenge.

If no domain is declared, if the proof cannot be verified, or if the proof is expired or malformed, `signals.domainProof.ok` MUST be `false`.

In baseline interoperability mode, implementations SHOULD support both of the following proof locations:

1. **DNS TXT**: a TXT record on `_hcs-28.<domain>` containing a token that starts with `hcs-28-domain-proof=`.
2. **HTTPS**: a GET request to `https://<domain>/.well-known/hcs-28-domain-proof.txt` whose response body contains a token that starts with `hcs-28-domain-proof=`.

The challenge token SHOULD bind the domain proof to the skill subject and publisher identity. A recommended token format is the following UTF-8 string:

```
hcs-28-domain-proof=<network>:<discovery_topic_id>:<skill_uid>:<version>:<account_id>:<expires_at>:<nonce>
```

Where:

- `account_id` is the HCS-26 publisher account id, if available;
- `expires_at` is an ISO-8601 timestamp in the future; and
- `nonce` is a verifier-provided random string.

If an `expires_at` value is present and is in the past, implementations MUST treat `signals.domainProof.ok` as `false`.

## Evidence (Recommended)

Implementations SHOULD preserve the verified domain and proof method, for example:

```json
{
  "domain": "example.com",
  "method": "dns_txt",
  "checkedAt": "2026-03-02T12:00:00.000Z",
  "expiresAt": "2026-04-01T12:00:00.000Z"
}
```
