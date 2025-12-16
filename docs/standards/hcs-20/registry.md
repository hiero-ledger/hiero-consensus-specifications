# Topic Registry

## Overview

The HCS Topic Registry introduces a standardized way to register Topic IDs on the Hedera Hashgraph and detail the protocol (HCS-20, etc). This registry is public and allows for the indexed search of registered topics that are both private and public (with and without submitKeys).

Version 0.0.5 of the desktop app has the UI to register a new topic id to the network.

**Public Topic ID for Registry:** 0.0.4362300

## Register Command

### JSON Structure

```json
{
  "p": "hcs-20",
  "op": "register",
  "name": "YOUR_TOPIC_NAME",
  "metadata": "OPTIONAL_METADATA (HIP-412 compliant)",
  "private": BOOLEAN_VALUE,
  "t_id": "TOPIC_ID_TO_REGISTER",
  "m": "OPTIONAL_MEMO"
}
```

### Fields

| Field    | Type    | Required | Description                    | Validation                           |
| -------- | ------- | -------- | ------------------------------ | ------------------------------------ |
| p        | String  | Yes      | Protocol identifier            | Must be 'hcs-20', case insensitive   |
| op       | String  | Yes      | Operation type                 | Must be 'register', case insensitive |
| name     | String  | Yes      | Name of the topic              | Max 100 chars, non-empty             |
| metadata | String  | No       | HIP-412 compliant metadata URL | Max 100 chars, valid URL             |
| private  | Boolean | Yes      | Privacy setting                | true/false                           |
| t_id     | String  | Yes      | Topic ID to register           | Must match Hedera account ID format  |
| m        | String  | No       | Additional memo                | Max 500 chars                        |

## Field Validation Details

### Protocol Identifier (p)

- Type: String
- Required: Yes
- Validation: Must be 'hcs-20' or future protocol numbers, case insensitive

### Operation Type (op)

- Type: String
- Required: Yes
- Validation: Must be 'register', case insensitive

### Name (name)

- Type: String
- Required: Yes
- Maximum length: 100 characters
- Must be non-empty

### Metadata (metadata)

- Type: String (URL)
- Required: No
- Maximum length: 100 characters
- Should be HIP-412 compliant
- Recommended format: `ipfs://bafyreid745vrmonapq677deya5gbnw33zx7f4ijqmten4fmxy5oxnwte5q/metadata.json`

### Privacy Setting (private)

- Type: Boolean
- Required: Yes
- Values:
  - `true`: Private topic
  - `false`: Public topic

### Topic ID (t_id)

- Type: String
- Required: Yes
- Format: Must match Hedera account ID format
- Validation regex: `^(0|(?:[1-9]\d*))\.(0|(?:[1-9]\d*))\.(0|(?:[1-9]\d*))$`

### Memo (m)

- Type: String
- Required: No
- Maximum length: 500 characters
- Purpose: Additional descriptions or notes

## Benefits

1. **Search-ability**

   - Topics are easily indexable
   - Enhanced discoverability
   - Searchable registry

2. **Standardization**

   - Uniform method for registration
   - Consistent topic management
   - HCS-20 standard compliance

3. **Transparency**
   - Clear topic management
   - Especially beneficial for public topics
   - Improved visibility

## Usage Notes

### Public Topics

- Set `private: false`
- Provide public topic_id
- Accessible to all users

### Private Topics

- Set `private: true`
- Ensure proper access control configuration
- Restricted access as needed
