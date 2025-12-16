---
title: "HCS-6: Dynamic Hashinals"
description: The HCS-6 standard defines a framework for creating dynamic Hashinals on the Hedera Hashgraph, enabling interactive and updatable on-chain assets.
sidebar_position: 6
---

# HCS-6 Standard: Dynamic Hashinals

### Status: Published

### Table of Contents

- [HCS-6 Standard: Dynamic Hashinals](#hcs-6-standard-dynamic-hashinals)
  - [Status: Published](#status-published)
  - [Table of Contents](#table-of-contents)
  - [Authors](#authors)
    - [Additional Authors](#additional-authors)
  - [Abstract](#abstract)
  - [Motivation](#motivation)
  - [Specification](#specification)
    - [Creating an HCS-6 Topic ID](#creating-an-hcs-6-topic-id)
    - [Metadata](#metadata)
    - [Submitting Messages](#submitting-messages)
    - [Validation](#validation)
    - [Limitations](#limitations)
    - [Conclusion](#conclusion)

## Authors

- Kantorcodes [https://twitter.com/kantorcodes](https://twitter.com/kantorcodes)

### Additional Authors

- Patches [https://twitter.com/tmcc_patches](https://twitter.com/tmcc_patches)

## Abstract

This standard is a sub-standard of [HCS-2](./hcs-2.md) which introduces a way to inscribe and tokenize **Hashinals** whose `metadata` can be updated at a whim by utilizing a non-indexed Topic ID. The latest message points to the current `metadata` for an NFT.

## Motivation

Creators of NFT Collections, sometimes desire the ability to change the metadata of an NFT, for example in cases of:

- correcting mistakes in metadata during initial inscription
- providing holders with new variations of artwork / attributes over time
- updating numerical stats from leveling up or down during a game

The list above is non-exhaustive and can be expanded to many use cases.

## Specification

HCS-6 is largely built on top of [HCS-2](./hcs-2.md) which describes how Advanced Topic Registries work. Please read that document for further clarity.

Creating a Dynamic **Hashinal** involves four steps.

1. Creating a valid, `non-indexed` [HCS-6](#creating-an-hcs-6-topic-id) Topic ID
2. Minting a new Serial Number on a Token ID following the [Metadata](#metadata) format
3. Creating a valid [HCS-1](./hcs-1.md) Topic ID
4. Submitting a valid message to the HCS-6 Topic ID with the HCS-1 file.

Steps 2-4 can be repeated every time the NFT will be updated.

### Creating an HCS-6 Topic ID

The `memo` field for Dynamic Hashinals must follow this format to be valid.

`hcs-6:non-indexed:{ttl}`

The only variable element in the memo would be the `ttl` field. We suggest a longer `ttl` as this is the time in seconds that gateways and clients will store the previous version of your `metadata` in their cache. In the future, gateways and clients may decide to prioritize Topics with longer `metadata` by imposing fees, introducing rate limits, etc. The minimum `ttl` must be `3600` (1 hour) to be valid, and the suggested minimum would be `86400` (1 day)

### Metadata

Dynamic **Hashinals** follow all of the same rules described in [HCS-5](./hcs-5.md), with one main exception being that they will utilize the `HCS-6` hcsStandard instead of `HCS-1`. The resulting [HRL](../definitions.md#hashgraph-resource-locator) minted onto a serial number includes the protocol number `6`

The format of the `metadata` on a dynamic **Hashinal** is as follows:

`hcs://6/{topicId}`

`topicId` is a valid HCS-6 Registry Topic ID in which data for this NFT will be or is written to.

The purpose of including the file standard in the HRL is to enable client applications to make logical decisions based on its existence. For example:

- a client might decide to use a different gateway to load immutable (HCS-5) versus Dynamic (HCS-6) Hashinals without needing to load the `memo` for each topic.
- analytics and indexer applications can search for all NFTs that include a HCS-6 HRL.

### Submitting Messages

In a `non-indexed` Topic ID, the most recent `register` operation is the current pointer for your metadata. You can submit a new message to update the `metadata` within the `ttl` specified on the `memo` of your HCS-6 Topic ID.

Each message follows this format.

```json
{
  "p": "hcs-6",
  "op": "register",
  // the HCS-1 Topic ID
  "t_id": "0.0.12345",
  "m": "Season 2 update for the Hashinal Hounds"
}
```

Clients, dApps, gateways, etc, will read the most recent sequence number's `t_id` to locate the current `metadata` of this Dynamic **Hashinal**

### Validation

Dynamic **Hashinals** are only valid when

- Their HCS-6 Topic ID is `non-indexed`
- The latest message in the HCS-6 Topic ID has a valid [HCS-1 Topic ID](./hcs-1.md) for the `t_id` field
- They specify a `ttl` that is at least `3600` (1 hour)
- It is created after `1713886229389232400` nanosecond timestamp

### Limitations

While HCS-6 is built on HCS-2, it includes some important limitations to improve UX.

1. Inscribed messages cannot utilize recursion in submitted messages to the registry. For example, the following would be invalid

```json
{
  "p": "hcs-6",
  "op": "register",
  // an HCS-6 Topic ID
  "t_id": "0.0.12345",
  "m": "Pointing metadata to another registry."
}
```

2. HCS-6 will always use a `non-indexed` Topic ID. `update` and `delete` operations will not be valid.

### Conclusion

HCS-6 clearly defines how to inscribe Dynamic Hashinals on the Hedera Hashgraph utilizing the Consensus Service. Note, to avoid all doubt, operations and functionality not written within this standard are invalid and should not be assumed.
