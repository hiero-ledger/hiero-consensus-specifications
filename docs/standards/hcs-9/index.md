---
title: "HCS-9: Poll Metadata"
---

#### [ hcs-9 - Poll Metadata Schema ]

**Status:** Published

**Authors:** May Chan

## Abstract

This standard introduces a metadata schema for polls to be used by the broader community.

Polls are a common method of community engagement in which a question is posed to voters. Voters then participate in the poll by voting on one or more options presented in the poll. Building out a poll framework on a public decentralized ledger such as Hedera adds increased transparency and trust in the voting process compared to solutions which rely on a database or server to record and report on the data.

Polls as a category are quite diverse, with each project wanting to run polls with their own rules, features and restrictions. This schema is designed with flexibility and modularity in mind, allowing for projects to extend the functionality to fit their purposes. A robust data architecture with standardized fields provides a structure for any platform to understand the basic poll data and display it.

This schema should be able to be implemented regardless of the exact implementation. Given an API endpoint or pointer to the data, any platform should be able to serve the information in this standard to third parties for standardized consumption.

This standard is issued alongside hcs-8: Poll Topics, which presents a standard for HCS Topic-based polls.

## Motivation

For polls to be decentralized, the data must be presented in an open standard that can be adopted by multiple platforms. This is challenging as polls are quite diverse and every platform has specific requirements for their polls.

Some scenarios for thought:

- Is each user only allowed one vote, or multiple? Is the number of votes constant across all users, or perhaps users receive votes for holding tokens on a certain date (a 'snapshot')?
- Is voting token-gated by a specific NFT, or perhaps a group of whitelisted NFTs? Or perhaps voting is gated by holding a certain quantity of a specific token? Perhaps there is a manually curated whitelist of users that are allowed to vote?
- Are users allowed to change their votes? Vote on multiple options? Cast negative votes?
- On the moderation side, perhaps moderators want the ability to change the end date, or update the post title, or change up the options. Or perhaps the creator wants to specify that no changes are allowed (an immutable poll).
- What about polls where the options are suggested and added by the users themselves?

A metadata standard should be flexible to encapsulate all these different rules, behaviours and features. At the same time, it should be understood that platforms will only develop and implement the features that they are specifically interested in. Most platforms are not interested in supporting the needs of other projects, which is why most voting platforms define their own metadata for their needs without bothering to create or adhere to a generalized standard.

However with standardized metadata come network effects that can enrich and add value to all voting platforms that implement them. The clearest example in the Hedera space is the NFT metadata standard HIP-412. With HIP-412 every NFT that is launched can be certain that it is minimally cross-compatible with launchpads, marketplaces, wallets and any other defi app. There may be features that an NFT has that are not supported by a platform (for example Augmented Reality features for 3D models, or music for music NFTs), but the basic information such as name, description and an image (that may be a placeholder) will be interpretable by any NFT platform on the network at no extra cost to the creator or the platforms.

There is a similar opportunity here with polls. At a minimum every poll has a question being posed and options to be voted on. Votes are non-fungible with equal weight, providing a final tally at the end to establish a clear winner. Polls and the activities of their voters can be aggregated or cross-posted between platforms, encouraging more community participation and providing rich possibilities for user engagement.

Furthermore, adoption of the standard allows for the community to pool development efforts and collaborate on open sourced implementation that can facilitate and accelerate development across the ecosystem as a whole.

## Rationale

The main purpose of this standard is to facilitate and reduce friction for community adoption around polls. In this way we can encourage adoption and enable community contributors to innovate and expand the feature set in a scalable manner.

This standard is written to be human-readable as much as possible, with data presented in JSON format per existing metadata implementations on Hedera.

Rules and behaviours are encapsulated in JSON object "modules". This standard defines several base modules, and establishes rules and guidelines for creation of additional modules. 
