---
description: The HCS-8 standard introduces a framework to use Hedera Consensus Service (HCS) topics to manage polls.
sidebar_position: 8
---

# HCS-8 Standard: Poll Topic Standard

**Status:** Published

**Authors:**

- May Chan
- Michael Kantor

## Abstract

This standard introduces a framework to use Hedera Consensus Service (HCS) topics to manage polls, following Hashgraph Online paradigms.

Polls are a common method of community engagement in which a question is posed to voters. Voters then participate in the poll by voting on one or more options presented in the poll. Building out a poll framework on a public decentralized ledger such as Hedera adds increased transparency and trust in the voting process compared to solutions which rely on a database or server to record and report on the data.

This framework provides a method to run a poll where poll activities are recorded on Hedera Hashgraph. Every poll is contained in a single HCS topic, allowing for retrieval of the history of the poll at any time.

This standard is issued alongside hcs-9: Poll Metadata Standards, which presents a robust specification for poll metadata for broader use in the ecosystem.

## Motivation

Using distributed ledger technology for voting and governance is a common topic in the Web3 community. On Hedera there have been several projects that have built voting platforms in order to address this niche. However, though the platforms themselves are quality platforms, they are often limited by the features which the specific projects are interested in. There does not, to the author's knowledge, exist a robust specification for voting built for broader community adoption.

The framework in this platform is built to be easily implementable by third parties. The standardized message and metadata format allows for any platform to take a poll topic ID and interact with the poll in a decentralized manner. Platforms which implement this framework will be cross compatible by default, only needing the topicID of a poll to retrieve all relevant information.

Furthermore, this framework is designed to fit a specific niche of community polls whose consequences are not tethered to legal frameworks or financial instruments. Design choices have been made that increase human readability and flexibility for poll creators, with fewer guards against user and platform error. The non-programmable nature of HCS, for example, leaves room for spam, errors in implementation, mistakes in counting, and other issues that other standards may protect against.

The design of this specification is such that for a properly implemented poll, all actions when properly implemented per the specification are auditable and trustless. These qualities are explored in greater detail in the following sections.

It should be noted that other voting frameworks exist on Hiero; notably HashioDAO (Smart contract based) and Creator Galaxy's CGIP-2 (HCS-topic based). These platforms offer different advantages that may make them more suitable for a user's specific needs. Links have been provided to them in the references section.

## Rationale

Utilizing the Hedera Consensus Service (HCS) allows for all data to be publicly readable, with verifiable timestamps and signatures from all participants.

Creating an HCS topic without a submit key allows for participation from any account on the network. For voting platforms which desire a completely decentralized, trustless voting process, this allows for participants to publicly sign off on votes without relying on any single entity - For example, even if the voting website blocked a voter, that voter could directly submit their HCS message to the network via a script and cast their votes.

It should be noted that using HCS in this way means there are no blockers to posting messages to the topic, and also no mechanisms to validate messages before being posted to the topic. Thus one trade-off of having an open topic is that all messages must be validated by platforms to ensure they are valid, when processing actions.

Invalid messages could be:

- Messages with incorrect formatting
- Messages that cast illegal votes per the rules of the poll
- Duplicate messages
- Spam messages

To avoid this, poll authors can create a HCS topic with a submit key. This locks participation only to entities that hold the submit key. The benefit of this approach is that all activities on the topic can be pre-validated by the author for validity prior to posting, allowing for a cleaner topic, reducing the validation work needed to be done by platforms reading the topic. This approach adds an element of centralization, since only the author can post messages to the topic they could censor the activities of certain accounts by choosing not to post messages from those accounts.
