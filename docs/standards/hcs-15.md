---
description: The HCS-15 standard provides a dynamic ability for account holders to create multiple instances of their accounts that use the same private key.
sidebar_position: 15
---

# HCS-15 Standard: Petals - Profile Accounts

### Status: Draft

### Version: 1.0

### Table of Contents

## Authors

- Patches [https://twitter.com/tmcc_patches](https://twitter.com/tmcc_patches)

### Abstract:

The HCS-15 standard provides a dynamic ability for account holders to create multiple instances of their accounts that use the same private key. This account dynamic enables multiple profiles, unique account ids, and asset holdings to be isolated in separate accounts on chain. 

The base account has the ability to orchestrate and execute transactions from each petal account as they are all utilizing the same private key.

### Motivation:

When utilizing HCS-11, the main memo of the account is restricted to one profile and one unique account id for that user or agent. Having the ability to have multiple profiles of HCS-11, tied to different unique identifiers, while also utilizing the same public/private key pair allows users and AI agents to dynamically create new identities and facilitate isolated escrows / commerce while maintaining the same signature.

## Specification

### Petal Account Creation

The initial account that a user creates is their base account. This base account is _highly recomended to be created with a ECDSA key + EVM Alias pair_. Creating the account this way will allow the base account to perform any complex EVM functionality that may require an accurate EVM Alias/private key pair (such as ECRECOVER).

Once the base account is created correctly, it can create it's first petal account.

Each Petal account contains the following attributes:

- (required) The same key as the base account
- (required) A HCS-11 profile memo (HCS-11 profile petal extension version) with valid inbound and outbound topic ids and base account reference defined
- (optional) A state topic for internal state consensus (see [HCS‑17](/docs/standards/hcs-17))

Petals may optionally contain and reference topics of information like state hashes.
Refer to [HCS‑17](/docs/standards/hcs-17) for state hashing on topics if additional topic functionality is needed for Petal accounts.

### Account Memo Structure

Petal accounts use the HCS‑11 memo convention to reference their profile document. Do not duplicate that standard here; instead, see [HCS‑11](/docs/standards/hcs-11) for the canonical grammar and examples.

- Memo format (delegated to HCS‑11): `hcs-11:<resource>`
- `<resource>` may be an HCS HRL (`hcs://<standard>/<topicId>`) or another supported URI (IPFS/Arweave/HTTPS) as defined by HCS‑11.

### Profile Schema Requirements

Petal accounts reuse the canonical [HCS‑11](/docs/standards/hcs-11) base profile schema. Refer to that standard for the authoritative field definitions.

Petal profiles shall populate the `base_account` field defined in HCS‑11 with the Hedera account ID of their controlling base account so integrators can link the shared private key lineage.

Below is a sample snippet of how this could be done with the JS SDK from hedera:

```ts
// 1. Generate the key‑pair that both new accounts will share
const sharedPrivKey = PrivateKey.generateECDSA(); // secp256k1
const sharedPubKey = sharedPrivKey.publicKey;
const evmAlias = sharedPubKey.toEvmAddress(); // 20‑byte, 0x‑prefixed

// 2. Create account #1 with the ECDSA key *and* EVM alias
const firstReceipt = await new AccountCreateTransaction()
  .setKey(sharedPubKey)
  .setAlias(evmAlias) // embeds the EVM address as the immutable alias
  .setInitialBalance(new Hbar(10)) // 10 ℏ so we can pay for next tx
  .execute(client)
  .then((tx) => tx.getReceipt(client));

// Not shown: Create and add HCS-11 profile

const firstAccountId = firstReceipt.accountId!.toString();
console.log(`🆗 First account: ${firstAccountId}`);

// 3. Create Petal Account by re‑using the same public key

const secondReceipt = await new AccountCreateTransaction()
  .setKey(sharedPubKey) // ★ same key as #1
  .setInitialBalance(new Hbar(1)) // tiny starting balance
  .setMaxAutomaticTokenAssociations(-1) // unlimited token airdrops
  .execute(client)
  .then((tx) => tx.getReceipt(client));

// Not shown: Create and add HCS-11 profile, referencing firstAccountId as base_account

const secondAccountId = secondReceipt.accountId!.toString();
console.log(`🆗 Second account (same PK): ${secondAccountId}`);
console.log(
  `🔑 Save this private key somewhere safe:\n${sharedPrivKey.toString()}`
);
```

## Security Concerns

- An alias set at creation cannot be changed later; if key‑rotation is a future requirement, create the account without a correct EVM alias

- Re‑using a key across multiple accounts provides diverse functionality but increases blast‑radius risk: compromise of one private key endangers every account that uses it: _plan your key‑management accordingly_.

- It is highly recommended to not rotate the keys of these accounts. If you must update the keys, be sure to rotate the petal account keys first, then the base account.

## Conclusion

The HCS-15 standard provides a new method of managing on-chain assets in a decentralized manner. The requirement of HCS-10 with HCS-11 profiles for petal accounts opens a wide range of utility for these accounts to be used across many instances.

Petal accounts unlock on-chain profiles for various uses. Applications can now requet users create a new petal account and have full isolated control over their application profiles (social profile, video game characters, etc), isolated assets for each profile, and unique profile identifiers (account ids) while also giving the user the ease of only needing one key to operate and sign all the transactions with.
