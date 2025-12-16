---
title: "HCS-13: Schema Registry"
description: This specification provides a standard way to define, register, and reference schemas for data validation and structure in the Hedera ecosystem, enabling type-safe data exchange and improved interoperability.
sidebar_position: 13
---

# HCS-13 Standard: Schema Registry and Validation

### Status: Draft

### Table of Contents

- [HCS-13 Standard: Schema Registry and Validation](#hcs-13-standard-schema-registry-and-validation)
  - [Status: Draft](#status-draft)
  - [Table of Contents](#table-of-contents)
  - [Authors](#authors)
    - [Primary Author](#primary-author)
  - [Abstract](#abstract)
  - [Motivation](#motivation)
  - [Specification](#specification)
    - [Architecture Overview](#architecture-overview)
    - [Schema Definition](#schema-definition)
      - [Schema Format](#schema-format)
      - [Schema Example](#schema-example)
    - [Schema Topics](#schema-topics)
    - [Message Registry Topics](#message-registry-topics)
      - [Registry Creation](#registry-creation)
    - [Schema Discovery Registry](#schema-discovery-registry)
      - [Discovery Registry Operations](#discovery-registry-operations)
        - [Register Schema](#register-schema)
        - [Deprecate Schema](#deprecate-schema)
    - [Schema Referencing](#schema-referencing)
      - [HRL Format](#hrl-format)
      - [Referencing in Messages](#referencing-in-messages)
      - [Referencing in NFTs](#referencing-in-nfts)
        - [Referencing a Schema in NFT Metadata](#referencing-a-schema-in-nft-metadata)
        - [Tokenizing a Schema](#tokenizing-a-schema)
    - [NFT Integration](#nft-integration)
      - [Benefits of Schema-Validated NFTs](#benefits-of-schema-validated-nfts)
      - [Example Use Cases](#example-use-cases)
    - [Schema Validation](#schema-validation)
    - [Schema Versioning](#schema-versioning)
      - [Version Management](#version-management)
      - [Backward Compatibility](#backward-compatibility)
    - [User Process for Schema Management](#user-process-for-schema-management)
    - [Schema Management Lifecycle](#schema-management-lifecycle)
    - [Conclusion](#conclusion)

## Authors

### Primary Author

- Kantorcodes [https://twitter.com/kantorcodes](https://twitter.com/kantorcodes)

## Abstract

The HCS-13 standard introduces a comprehensive framework for defining, registering, and referencing schemas within the Hedera ecosystem. By providing a standardized approach to schema management, this specification enables type-safe data exchange, improved interoperability, and enhanced validation capabilities across decentralized applications.

## Motivation

As the Hedera ecosystem grows, the need for structured, validated data becomes increasingly important. Without a standardized schema system, developers face challenges in ensuring data consistency, validating message formats, and enabling interoperability between applications. The HCS-13 standard addresses these challenges by providing a robust schema registry system that leverages the Hedera Consensus Service (HCS) to store, manage, and reference schemas in a decentralized manner.

Key motivations include:

1. Enabling type-safe data exchange between applications
2. Providing a standardized way to validate message structures
3. Improving interoperability through shared schema definitions
4. Supporting schema evolution and versioning
5. Enhancing NFT metadata with structured, validated data
6. Creating a foundation for advanced data processing and analytics

## Specification

The HCS-13 standard builds upon the [HCS-2](./hcs-2.md) registry system and introduces specific mechanisms for schema management. It defines how schemas are created, registered, referenced, and used for validation.

### Architecture Overview

The HCS-13 standard involves three distinct types of topics:

1. **Schema Topics**: Topics that store JSON Schema definitions as HCS-1 files. Each schema is stored in an HCS-1 topic, and versioning is managed through HCS-2 operations.
2. **Message Registry Topics**: Topics that store validated messages conforming to a specific schema.
3. **Schema Discovery Registry**: An HCS-2 guarded registry that enables discoverability of schemas across the ecosystem.

This architecture allows for clear separation of concerns while enabling powerful schema validation and discovery capabilities.

### Schema Definition

Schemas in HCS-13 are defined using JSON Schema, a widely adopted standard for describing JSON data structures. JSON Schema provides a rich set of validation rules and is supported by numerous libraries across programming languages.

#### Schema Format

A schema definition must be a valid JSON Schema document (draft-07 or later) and should be stored as an HCS-1 file. The schema must include:

1. A `$schema` property indicating the JSON Schema version
2. A `title` property providing a human-readable name
3. A `description` property explaining the schema's purpose
4. A `type` property specifying the root data type

Additional metadata may be included to provide context and documentation.

#### Schema Example

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "UserProfile",
  "description": "Schema for user profile data",
  "type": "object",
  "properties": {
    "username": {
      "type": "string",
      "minLength": 3,
      "maxLength": 50
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "age": {
      "type": "integer",
      "minimum": 13
    },
    "interests": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  },
  "required": ["username", "email"]
}
```

### Schema Topics

Schema topics store the actual JSON Schema definitions as HCS-1 files. The workflow for creating and managing a schema topic is:

1. Create an HCS-1 file containing the JSON Schema definition
2. Create an HCS-2 topic to manage versions of the schema
3. Register the HCS-1 file in the HCS-2 topic using the register operation

This approach leverages:

- HCS-1 for storing the schema content (which may exceed the 1KB message limit)
- HCS-2 for versioning and managing references to the schema

Example schema registration in an HCS-2 topic:

```json
{
  "p": "hcs-2",
  "op": "register",
  "t_id": "0.0.123456", // Topic ID of the HCS-1 file containing the schema
  "metadata": {
    "name": "UserProfile",
    "description": "Standard user profile schema"
  },
  "m": "Initial schema definition"
}
```

### Message Registry Topics

Message registry topics store validated messages that conform to a specific schema. These topics use a modified HCS-2 format with a memo that references the schema topic ID.

#### Registry Creation

A message registry topic is created as an HCS topic with a specific memo format that indicates its purpose and configuration:

`hcs-13:[indexed]:[ttl]:[schemaTopicId]`

| Field           | Description                                                                            | Example Value |
| --------------- | -------------------------------------------------------------------------------------- | ------------- |
| `hcs-13`        | Indicates this is a message registry following the HCS-13 standard                     | `hcs-13`      |
| `indexed`       | Enum value (0 or 1) indicating if all messages need to be processed or only the latest | `0`           |
| `ttl`           | Time-to-live in seconds for caching purposes                                           | `86400`       |
| `schemaTopicId` | Topic ID of the HCS-2 topic managing the schema                                        | `0.0.123456`  |

Example memo: `hcs-13:0:86400:0.0.123456`

### Schema Discovery Registry

The schema discovery registry is an HCS-2 guarded registry that enables discoverability of schemas across the Hedera ecosystem. This registry allows developers to publish their schemas and make them available for others to use.

#### Discovery Registry Operations

The schema discovery registry supports operations for managing schema discoverability.

##### Register Schema

To register a schema in the discovery registry:

```json
{
  "p": "hcs-13",
  "op": "register",
  "t_id": "0.0.123456", // Topic ID of the HCS-2 topic managing the schema
  "metadata": {
    "author": "0.0.789012",
    "description": "Standard user profile schema",
    "tags": ["profile", "user", "identity"]
  },
  "m": "Initial schema registration"
}
```

##### Deprecate Schema

To mark a schema as deprecated in the discovery registry:

```json
{
  "p": "hcs-13",
  "op": "deprecate",
  "t_id": "0.0.123456", // Topic ID of the schema being deprecated
  "replacement": "0.0.789012", // Optional Topic ID of replacement schema
  "metadata": {
    "reason": "Security vulnerability in validation logic"
  },
  "m": "Schema deprecated due to security concerns"
}
```

### Schema Referencing

Schemas can be referenced in various contexts using Hedera Resource Locators (HRLs).

#### HRL Format

The HRL format for referencing a schema follows the pattern:

`hcs://13/[topicId]`

Where `topicId` is the topic ID of the HCS-2 topic managing the schema.

To reference a specific version of a schema, append a fragment identifier with the sequence number:

`hcs://13/[topicId]#[sequenceNumber]`

Examples:

- `hcs://13/0.0.123456` - References the latest version of the schema
- `hcs://13/0.0.123456#42` - References version 42 of the schema (sequence number 42)

#### Referencing in Messages

When a message conforms to a specific schema, it can include a reference to that schema:

```json
{
  "p": "hcs-13",
  "op": "data",
  "schema": "hcs://13/0.0.123456",
  "t_id": "0.0.789012", // Topic ID of the HCS-1 file containing the data
  "m": "User profile data"
}
```

For smaller data that fits within the 1KB limit, the data can be included directly:

```json
{
  "p": "hcs-13",
  "op": "data",
  "schema": "hcs://13/0.0.123456",
  "data": {
    "username": "alice",
    "email": "alice@example.com",
    "age": 25,
    "interests": ["blockchain", "programming"]
  },
  "m": "Inline user profile data"
}
```

#### Referencing in NFTs

There are two distinct ways to use schemas with NFTs:

1. **Reference a schema in NFT metadata**: Include a schema reference in the NFT's metadata to indicate the structure of its attributes
2. **Tokenize a schema**: Use the schema's HRL as the NFT's metadata to create a token representing the schema itself

##### Referencing a Schema in NFT Metadata

When creating an NFT with structured attributes, include a schema reference in the metadata:

```json
{
  "name": "Digital Identity Card",
  "description": "A digital identity card following the standard schema",
  "image": "hcs://1/0.0.456789",
  "schema": "hcs://13/0.0.123456",
  "attributes": {
    "username": "bob",
    "email": "bob@example.com",
    "age": 30,
    "interests": ["art", "music", "technology"]
  }
}
```

##### Tokenizing a Schema

To create an NFT that represents a schema itself, use the schema's HRL as the NFT's metadata:

```
hcs://13/0.0.123456
```

### NFT Integration

The HCS-13 standard enhances NFTs with structured, validated data, enabling type-safe attributes and improved interoperability.

#### Benefits of Schema-Validated NFTs

1. **Data Validation**: Attributes conform to a defined structure with validation rules
2. **Type Safety**: Applications can rely on consistent attribute formats
3. **Enhanced Metadata**: Support for complex, nested data structures
4. **Interoperability**: Different applications can process NFT data consistently

#### Example Use Cases

- **Gaming Items**: Define schemas for different item types with specific attributes
- **Digital Identity**: Create verifiable credentials with standardized fields
- **Collectibles**: Ensure consistent attribute formats across collections
- **Real Estate**: Standardize property metadata for marketplace interoperability

### Schema Validation

The validation process involves:

1. Retrieving the schema from the referenced HCS-1 file (via the HCS-2 topic)
2. Parsing the schema as a JSON Schema document
3. Retrieving the data to be validated
4. Validating the data against the schema
5. Reporting validation results

Applications should validate data against schemas before storing it to ensure only valid data is registered.

### Schema Versioning

Schema versioning is handled through the HCS-2 topic that manages references to the schema HCS-1 files.

#### Version Management

Each time a schema is updated:

1. Create a new HCS-1 file with the updated schema
2. Submit a new register operation to the schema's HCS-2 topic referencing the new HCS-1 file
3. The new message becomes the current version of the schema
4. Previous versions remain accessible through their sequence numbers

Example schema update:

```json
{
  "p": "hcs-2",
  "op": "register",
  "t_id": "0.0.567890", // Topic ID of the new HCS-1 file containing the updated schema
  "metadata": {
    "name": "UserProfile",
    "description": "User profile schema with profile picture support"
  },
  "m": "Added optional profile picture field"
}
```

#### Backward Compatibility

When updating schemas, developers should consider backward compatibility:

- Adding optional fields preserves compatibility
- Removing required fields breaks compatibility
- Relaxing validation rules preserves compatibility
- Tightening validation rules may break compatibility

### User Process for Schema Management

The typical workflow for schema management includes:

1. **Define Schema**: Create a JSON Schema document
2. **Store Schema**: Store the schema as an HCS-1 file
3. **Create Schema Topic**: Create an HCS-2 topic to manage schema versions
4. **Register Schema**: Register the HCS-1 file in the HCS-2 topic
5. **Create Message Registry**: Create a message registry topic referencing the schema topic
6. **Register in Discovery Registry**: Register the schema in the discovery registry
7. **Prepare Data**: Create data conforming to the schema
8. **Validate Data**: Validate the data against the schema
9. **Store Data**: Store the data as an HCS-1 file (if needed)
10. **Register Data**: Register the data in the message registry

### Schema Management Lifecycle

The following diagram illustrates the complete lifecycle of schema management using the HCS-13 standard:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Define Schema  │────▶│ Store Schema as │────▶│Create Schema    │
│  (JSON Schema)  │     │   HCS-1 File    │     │Topic (HCS-2)    │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                                                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│Create Message   │────▶│Register in      │────▶│  Prepare Data   │
│Registry Topic   │     │Discovery Registry│     │                 │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                                                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Update Schema  │◀────│ Register Data   │◀────│ Validate Data   │
│  (As Needed)    │     │ in Registry     │     │ Against Schema  │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

This lifecycle demonstrates how schemas are created, stored, registered, and evolved over time:

1. First, a schema is defined using JSON Schema format
2. The schema is stored as an HCS-1 file due to potential size limitations
3. An HCS-2 topic is created to manage versions of the schema
4. A message registry topic is created to store data conforming to the schema
5. The schema is registered in the discovery registry for discoverability
6. Data is prepared, validated against the schema, and registered
7. When updates are needed, a new version of the schema is created and registered

### Conclusion

The HCS-13 standard provides a comprehensive framework for schema management in the Hedera ecosystem. By leveraging HCS-1 for content storage and HCS-2 for versioning, it creates a robust system for defining, registering, and referencing schemas.

Key benefits of the HCS-13 standard include:

1. **Type-safe data exchange** between applications
2. **Standardized validation** of message structures
3. **Improved interoperability** through shared schema definitions
4. **Flexible versioning** that maintains backward compatibility
5. **Enhanced NFT metadata** with structured, validated data

By following this standard, developers can create more reliable, interoperable applications that leverage the power of structured data validation within the Hedera ecosystem.
