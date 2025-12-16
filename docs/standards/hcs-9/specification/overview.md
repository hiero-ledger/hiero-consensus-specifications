---
title: Overview
sidebar_position: 1
---

#### [ hcs-9 - Poll Metadata Schema ]

## Specification Overview

### Schema Philosophy 

1. **Object-based Modularity:** The main mechanism to facilitate feature extensibility is through the use of JSON objects as modules to define the rules of the poll. This allows features to be plug-and-play, while providing mechanisms for developers to define, document and collaborate on them.

2. **Standardized Interfaces:** The data is structured to allow apps to explicitly identify the "format" being used in the metadata, allowing apps to seamlessly interact and read supported formats and gracefully fail if a particular format is not supported. 

3. **Strictly Defined Baseline:** When a format is not supported by a platform, it does not understand the ruleset and thus does not have the capability to actively interact with the poll. However, the data is structured in such a way that any platform can read the metadata and display the poll and its results, acting as a passive viewer and displaying at least a baseline minimum amount of data for a user.

### JSON Schema Formatting

This standard adopts JSON Schema principles. JSON Schema provides a human-readable way to describe the expected format of JSON documents, allowing for automated validation and testing and support interoperability. This format also serves as a machine-readable documentation of the data structure, making it easier for developers to understand and implement the standards. Furthermore, JSON Schema includes mechanisms for versioning, allowing clearer communication of changes and easier backwards compatibility management.

The intent is to provide robust, clear and developer-friendly approaches to these data structures. This will lead to more reliable implementations, reduced development time, and improved overall quality of systems developed on the standard.

The JSON Schema files are all provided here: [Schema Definitions](/docs/standards/hcs-9/schema-definitions)

## Base Schema, Vote Schema and Module Schema

This standard consists of three major components:

1. The Base Schema defines a poll's properties and rules.
2. Module Schemas define the structure of each of the modules in the base schema.
3. The Vote Schema defines voting actions.

### Base hcs-9 Schema Specification

This is the base schema that forms the root of the poll. 

The components of the schema are described below:

```
{
  "schema" - Strictly defines the schema. Must be "hcs-9"
  "title" - The title of the poll, the question being asked
  "description" - The description of the poll
  "author" - The author of the poll. For HCS implementations this must be the Hedera account Id of the author
  "votingRules" - A votingRules module, defines the rules around voting
  "permissionsRules" - A permissionsRules module, defines the list of users who are allowed to participate
  "manageRules" - A manageRules module, defines the rules for managing the status of the poll
  "updateRules" - An updateRules module, defines the rules for updating the poll information and settings
  "options" - An options module, defines the options that are being voted on
  "status" - The status of the poll. Must be one of several pre-defined states.
  "startDate" - When the poll is scheduled to start
  "endConditionRules" - An endConditionRules module, which defines the end condition
  "custom-parameters" - A container for any additional custom parameters if desired
}
```

The base schema is explored in detail here: [Base Schema Specification](base-schema)

### Module Schema Specification

The base schema is a container for the modules which define the behaviour and rules of the poll. Each module manages a specific scope of the poll:

**votingRules** defines *how many* votes each user receives

**permissionsRules** defines *who* is allowed to participate in voting

**manageRules** defines the rules around managing the *status* of the poll and *who* is allowed to manage it.

**updateRules** defines the rules around updating the poll information and settings and *who* is allowed to update them

**endConditionRules** defines the rules around *what triggers the end of the poll*

Each of these modules will be explored in the upcoming sections.

#### Module Extensibility

The structure of this schema has specifically been designed to allow for developers to extend the functionality to suit their specific use case. Each module is defined under the scope of 'hcs-9', however it is intended that if some voting feature or behaviour is not captured in this specification, that a developer can create their own JSON Schema module and implement it on their own voting platform by following the structure outlined in this standard.

For example, suppose Voting Site Alpha would like to define a specific permissionsRules module that calls their AlphaAPI to receive the list of users who are allowed to vote on their poll. A new permissionsRules module can be defined by the Alpha team and plugged into with the permissionsRules schema field defined as "schema":"alpha" instead of "schema":"hcs-9". They can then build their platform to use their schema.

In this scenario, suppose there is Poll Aggregator Beta, a website that aggregates polls. It only supports hcs-9 compliant polls, and also allows users to vote on open polls or even manage polls that they're the author of. Beta can read the Alpha team's base poll schema since it follows hcs-9, however when it parses the permissionsRules module it does not recognize the 'alpha' schema. This means that Beta cannot determine who is allowed to vote on the poll, which means it does not have the capability of allowing a user to vote on the poll.

However, because the poll still follows the hcs-9 schema, Beta can still display the information of the poll - What the title is, what the options are, how many votes have been placed on the poll. When the poll ends they can display the winning option.

Thus, the modularity allows team Alpha to build out specific functionality that it needs for its usecase, while still allowing team Beta (and other teams) to aggregate its data and extend visibility of their polls to the broader ecosystem.

Furthermore, if Alpha also provides a detailed public specification of their schema, Beta could implement the schema on their site and support the full functionality on Alpha's polls.

### Vote Schema Specification

When a vote is placed on the poll, the data must follow the vote schema defined here.

The vote schema is very simple and is optimized to provide the minimum amount of information to vote on a poll.

```
{
  “accountId” - The account id of the voter
  “optionId” - The unique id of the option
  “weight” - The weight of the vote
}
```

We define a **vote** as the action of indicating a user's choice in the poll, whereas **weight** is used as a measure of the magnitude of a user's vote action. The terms are used to help distinguish these two concepts, as english tends to use 'vote' as both a verb and also a noun.

For an hcs-9 poll, the winning option is the one with the most weight allocated to it by user votes when the end condition is met.

#### Regarding the validity of a vote

Validation is done by the server (and also, ideally, by the front end that the user is using as well), so the legality of a vote is not part of the scope of placing a vote. 

Authentication of the user is also not part of this schema and is assumed to be done by the platform 'above' the metadata layer. In the case of the sister standard hcs-8, which defines poll topics run via the Hedera Consensus Service, the accountId in the vote metadata here must also be the account that signs the HCS message.
