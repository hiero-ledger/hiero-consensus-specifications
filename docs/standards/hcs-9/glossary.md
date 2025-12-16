---
title: Glossary
sidebar_position: 7
---

## General Terms

#### module

Modules are the modular JSON objects that define the functions and behaviour of the poll. Each module has its own scope - for example, the vote-rules module establishes the rules around voting actions. This standard has defined several standard modules, but the intent is for developers to write their own modules and plug them into the base schema to extend the functionality to suit their specific needs.

More detailed information is in the documentation: [Module Schema](./specification/module-schema.md)

#### vote

A **vote** is the action of indicating a user's choice in the poll. This term is used alongside *weight* to help distinguish between the action and the 'amount' that a user contributes to an option, as english tends to use 'vote' as both a verb and also a noun. 

A user can vote (verb) on the poll with a weight or vote-weight of 10. Colloquially we can say that a user placed ten votes (noun) on the poll.

#### weight

**weight** or vote-weight is the magnitude of a user's vote action. Weight is fungible, so the weight of every user's votes can be simply added up to get the final total weight that has been placed on an option. The winning option of a poll is defined as the option that has received the most weight.

## Tags

Tags are used in this specification to provide concise information about the data.

#### case-insensitive

These fields should be validated in a case-insensitive way. 'OPEN' is equivalent to 'open' and 'OpEn'. 

It is recommended that developers convert strings to lower-case when retrieving data in the schema.

#### optional

Optional fields can be missing from the data. Apps should handle the behaviour when the data is present or provide a 'default' behaviour when it is missing. The specification describes the default behaviour for each optional field.

#### required

Required fields must be defined. If not then validation should fail.

#### snapshot

A snaphot is a record of the accounts which hold a particular token, recording balance and for NFTs, serial numbers in the account's possession at the time of the snapshot. 

Snapshots are important when resolving eligibility for actions as they establish a reference point for which to make decisions. Whereas, if no snapshot were taken but instead a user's current balance was taken at the time of an action, a user can manipulate the system by taking actions then transferring the assets to another account, which could then take actions.

#### unique

Unique fields act as indexes or identifiers in the data and must be unique amongst other objects of the same type.