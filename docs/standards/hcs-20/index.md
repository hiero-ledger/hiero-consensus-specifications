---
description: HCS-20 is a new proposed standard (created by @TurtelMoonCC & @HGraphPunks) that defines how points can be managed and audited on the Hedera Hashgraph. It is inspired by the BRC-20 protocol on Ordinals and has extended the functionality to auditable points in addition to introducing inscriptions on Hedera / Hashinals.
---

# HCS-20 Standard: Auditable Points Standard on Hedera

### Status: Published

- [HCS-20 Standard: Auditable Points Standard on Hedera](#hcs-20-standard-auditable-points-standard-on-hedera) - [Authors](#authors)
  - [Solving the Auditable Points Challenge](#solving-the-auditable-points-challenge)
  - [The Role of Hedera](#the-role-of-hedera)
    - [Hedera's Main Features:](#hederas-main-features)
  - [Harnessing HCS-20 with the Turtle Moon HCS App](#harnessing-hcs-20-with-the-turtle-moon-hcs-app)
  - [Two Operational Modes](#two-operational-modes)
    - [Main public topic Id for inscriptions:](#main-public-topic-id-for-inscriptions)
  - [Operations and State Calculation](#operations-and-state-calculation)
    - [Key Operations](#key-operations)
    - [Validity of Transactions](#validity-of-transactions)
  - [Conclusion](#conclusion)

#### Authors

- Patches [https://twitter.com/TMCC_Patches](https://twitter.com/TMCC_Patches)

**HCS-20** is a new proposed standard (created by @TurtelMoonCC & @HGraphPunks) that defines how points can be managed and audited on the Hedera Hashgraph. It is inspired by the BRC-20 protocol on Ordinals and has extended the functionality to auditable points in addition to introducing inscriptions on Hedera / Hashinals.

## Rationale

The HCS-20 standard emerges as a solution to a pressing problem in the web2 and web3 world: the need for a transparent, auditable, and versatile control system for points. Its primary benefits include:

- **Public Audit-ability**: Creates audit log for transparency in point transactions.
- **No LP or Value Expectation Issues**: Streamlines operations without liquidity, compliance, or value complications. Great for loyalty points, gaming points, etc.
- **Control Flexibility**: If in private mode, it offers the creator the power to oversee all point movements.

> **WARNING**
> THIS IS IN ALPHA
> The Indexer will be updated periodically and the way transactions are calculated may change as the standard matures.
> No points are intended to have any value.

## Standard Details

## Idea

The HCS-20 standard is designed to leverage the Hedera Consensus Service (HCS) for creating, managing, and transferring auditable ownership of points. It utilizes topic IDs on the Hedera Hashgraph to record JSON-encoded transactions, providing a framework for a auditable pointing system.

### Private (✅ Submit Key)

A topic ID with a submit key allows control of all data posted to that topic. This is great for most use cases like gaming, leaderboard points, loyalty points, etc.

**Valid transaction indexing note:**
Indexers of topic Ids with submit keys shouldn't require the payer account id to be the same as the one moving balances. Since the submit key requirement implies only authorized parties should have write access to move funds, all transfer transactions written are authorized.

### Public (❌ No Submit Key)

A topic ID with no submit key allows anyone to sign and write transaction openly to the public topic Id. Creators can make their own topic ids, or can use the proposed main topic id.

Benefits of using the main public topic Id is to give access to a central location of all inscriptions for HCS-20 public points where indexers don't need to index multiple topic Ids for balances.

**Valid transaction indexing note:**
Indexing valid balances also includes checking if the payer account id is the same as the transfer of balance out of an account. If payer account is different than the account having points removed, it's indexed as a failed transaction.

**Proposed main public topic Id:** 0.0.4350190

## JSON Structure

### Deploy Points

```json
{
  "p": "hcs-20",
  "op": "deploy",
  "name": "point_name",
  "tick": "unique_point_identifier",
  "max": "max_supply",
  "lim": "optional_limit_of_mint_per_transaction",
  "metadata": "optional_metadata",
  "m": "optional_memo"
}
```

| Key      | Required | Description                                                       |
| -------- | -------- | ----------------------------------------------------------------- |
| p        | Yes      | Protocol identifier, specifies HCS-20                             |
| op       | Yes      | Operation type, here it's 'deploy'                                |
| name     | Yes      | Name of the point, describes the asset being created              |
| tick     | Yes      | Unique identifier for the point, akin to a ticker symbol          |
| max      | Yes      | Maximum supply of the point, sets the upper limit                 |
| lim      | No       | Limit per transaction for minting, optional                       |
| metadata | No       | Optional additional data related to the points (HIP-412 standard) |
| m        | No       | Optional additional memo related to the operation                 |

### Mint Points

```json
{
  "p": "hcs-20",
  "op": "mint",
  "tick": "unique_point_identifier",
  "amt": "number_of_points",
  "to": "recipient_hedera_address",
  "m": "optional_memo"
}
```

| Key  | Required | Description                                          |
| ---- | -------- | ---------------------------------------------------- |
| p    | Yes      | Protocol identifier, specifies HCS-20                |
| op   | Yes      | Operation type, here it's 'mint'                     |
| tick | Yes      | Unique identifier of the point to be minted          |
| amt  | Yes      | Amount of points to mint                             |
| to   | Yes      | Address of the recipient receiving the minted points |
| m    | No       | Optional additional memo related to the operation    |

### Burn Points

```json
{
  "p": "hcs-20",
  "op": "burn",
  "tick": "unique_point_identifier",
  "amt": "number_of_points",
  "from": "holder_hedera_address",
  "m": "optional_memo"
}
```

| Key  | Required | Description                                             |
| ---- | -------- | ------------------------------------------------------- |
| p    | Yes      | Protocol identifier, specifies HCS-20                   |
| op   | Yes      | Operation type, here it's 'burn'                        |
| tick | Yes      | Unique identifier of the point to be burned             |
| amt  | Yes      | Amount of points to burn                                |
| from | Yes      | Address of the holder from whom points are being burned |
| m    | No       | Optional additional memo related to the operation       |

### Transfer Points

```json
{
  "p": "hcs-20",
  "op": "transfer",
  "tick": "unique_point_identifier",
  "amt": "number_of_points",
  "from": "sender_hedera_address",
  "to": "recipient_hedera_address",
  "m": "optional_memo"
}
```

| Key  | Required | Description                                       |
| ---- | -------- | ------------------------------------------------- |
| p    | Yes      | Protocol identifier, specifies HCS-20             |
| op   | Yes      | Operation type, here it's 'transfer'              |
| tick | Yes      | Unique identifier of the point to be transferred  |
| amt  | Yes      | Amount of points to transfer                      |
| from | Yes      | Address of the sender                             |
| to   | Yes      | Address of the recipient                          |
| m    | No       | Optional additional memo related to the operation |

## Field Validation

### Common Fields

- **Protocol Identifier (p)**

  - Type: String
  - Required: Yes
  - Validation: Must be 'hcs-20', case insensitive

- **Operation Type (op)**

  - Type: String
  - Required: Yes
  - Validation: Must be one of: 'deploy', 'mint', 'burn', 'transfer', 'register'

- **Number Fields (amount, max, lim)**

  - Type: String representing a number
  - Length: ≤ 18 characters
  - Validation: Must be valid number, defaults to 0 if conversion fails

- **Account Fields (toAddress, fromAddress, topicId)**
  - Type: String
  - Pattern: ^(0|(?:[1-9]\d*))\.(0|(?:[1-9]\d*))\.(0|(?:[1-9]\d\*))(?:-([a-z]{5}))?$
  - Note: If hyphen present, only part before is considered

### Operation-Specific Validation

- **Register Operation**

  - Name: 1-100 characters
  - Topic ID: Uses account field validation
  - Metadata: Optional string

- **Deploy Operation**

  - Tick: Lowercase, trimmed string
  - Name: 1-100 characters
  - Max: Number field, defaults to Infinity
  - Lim: Number field, defaults to Infinity

- **Mint/Burn/Transfer Operations**
  - Tick: Lowercase, trimmed string
  - Amount: Number field validation
  - Addresses: Account field validation

## State Calculation

- Deployments initialize point contract without affecting state
- Mints add to recipient's balance
- Transfers adjust sender and receiver balances
- Valid transfers require sufficient balance
- Valid mints must not exceed max supply or lim
- Transaction order determined by consensus timestamp

## Harnessing HCS-20 with the Turtle Moon HCS App

The Turtle Moon App simplifies the creation and management of HCS-20 points. Here's how you can leverage its features:

- **Download and Install**: Available for Mac, Windows, and Linux. [Install here](https://patches-1.gitbook.io/hcs-20-auditable-points/download).
- **Create and Interact**: Easily initiate a new topic, add wallet and topic ID details, and create JSON for HCS-20 points.
- **Submit and Read**: Deploy your points and track point balances through topic ID indexing.

## Two Operational Modes

- **Private Mode with Submit Key**: This mode offers control over data posted to a topic, ideal for most use cases where authorized parties require exclusive write access.
- **Public Mode without Submit Key**: Allows open, public transactions, suitable for decentralized applications where transparency is key, and open mints are wanted.

### Main public topic Id for inscriptions:

- `0.0.4350190`

## Operations and State Calculation

HCS-20 standardizes operations like deploying, minting, burning, and transferring points. The state of these points is determined by aggregating these operations on the HCS topic IDs.

### Key Operations

- **Deploy Point Contract**: Initialize a new asset.
- **Mint Point**: Generate and assign new points.
- **Burn Point**: Remove points from circulation.
- **Transfer Point**: Facilitate point movement between accounts.

### Validity of Transactions

Transactions are valid based on several criteria, including the balance availability at the time of inscription and the consensus timestamp order. For public mode operations, the payer account ID must match the account from which points are removed.

## Conclusion

The HCS-20 standard, complemented by the Turtle Moon App, offers a robust framework for creating an auditable point systems on Hedera Consensus Service Topic IDs.
