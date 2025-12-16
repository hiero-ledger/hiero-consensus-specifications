---
title: Specification
sidebar_position: 1
---

#### [ hcs-8 - Poll Topic Standard ]

### Framework Philosophy

1. **Single Topic:** In this framework a poll is completely encapsulated in a single HCS topic. The first message in the topic (the Register message) declares all the settings for the poll, including rules for voting and updating the poll. Critically, the Register message defines the set of legal actions, which is crucial to allow for automated tabulation of scores, and dismissal of illegal actions and spam.

2. **Human-Readable:** Action metadata is designed to be human-readable. Where possible, the contents of a message are to be sent to HCS in plain-text, allowing for users to read the actions in the poll via a Hedera explorer. The notable exception to this rule is the register message which may point to an external URI, since poll metadata will likely be larger than the character limit of a single HCS message.

3. **Third-Party Auditable:** All actions are posted to HCS and signed by the appropriate parties, allowing neutral third parties to audit the voting activity during and after the poll. Since there is no trustless method for programmatic validation with a pure HCS support, it is imperative that the rules are clearly defined and then followed to maintain the integrity of the voting results.

4. **Discard incorrectly constructed messages:** Messages that do not follow the Message Format precisely are ignored as a policy.

5. **Messages are read in order:** Polls are read in chronological order from oldest to newest. This is also important when the update operation is used to change the settings of the poll. 

6. **Modularity:** The framework is set up to be robust but flexible. hcs-8 defines the methods to take action on polls, but relies on the metadata structure of the hcs-9 standard to define the specifics. This modularity should allow for projects to innovate on poll types while following the same structure to maximize portability.

7. **Legal vs Illegal Actions:** hcs-8 does not explicitly define legal and illegal actions, but leaves it to the metadata standard to define and lay out rules. This is done to separate the format and flow of the poll topic from the behaviour of the poll.

8. **Searchability:** To faciliate searchability of hcs-8 topics and to adhere to other HCS standards, the topic creation transaction must follow the prescribed memo format.

Scenario 1:

| Id | Operation | Action                                       |
|----|-----------|----------------------------------------------|
| 0  | register  | Register the poll, initiating it as 'closed' |
| 1  | vote      | Vote for option 1                            |
| 2  | manage    | Open poll                                    |


In this scenario, a vote operation is placed when the poll is closed, and thus would be discarded.

Scenario 2:

| Id | Operation | Action                                       |
|----|-----------|----------------------------------------------|
| 0  | register  | Register the poll, setting the poll to one vote per person, with no permission to change votes. |
| 1  | manage      | Open Poll  |
| 2  | vote (by Bob)    | Vote for option 1 |
| 3  | vote (by Bob)    | Vote for option 2 |

In this scenario, Bob has one vote allocated to them. At the time of message 2, Bob places their vote for option 1, which is valid. At the time of message 3, Bob no longer has any votes allocated to them so the vote for option 2 is illegal and thus ignored when tallying votes.

### Topic Set Up

The parameters passed when creating the HCS topic for the poll affect the behaviour of the poll.

#### SubmitKey

Authorizes sending messages to this topic.

**Option 1: Do not specify a submit key**

This allows anyone to post messages to the poll, ensuring that the poll creator has no control over a voter's ability to vote. This keeps the poll completely open with any party able to read and write to the topic.

**Option 2: Specify a submit key**

This allows only the creator (and others who are given the submit key) the ability to add messages to the poll. This can be desirable to prevent spam from clogging up the message history. It can also prevent users from (either accidentally or on purpose) declaring illegal actions such as voting multiple times in a single-choice poll.

#### Admin Key

Authorizes update topic and delete topic transactions

#### Topic Memo

Sets a short memo that is stored with the topic (100 bytes)

This memo MUST be exactly 'hcs-8' (case sensitive). This is to adhere to the format of other HCS standards and allow for future extensibility through enums.

Valid Memos: 'hcs-8'

Invalid Memos: '' (empty), 'poll type: hcs-8', 'hcs8', 'HCS-8', 'hcs-8:hedera poll', 'hcs-8 community voting topic'

Note to avoid ambiguity: This is explicitly related to the memo on the topic create transaction. Topic messages and metadata memos do not need to follow the instructions in this section.

### Message Format and Usage

Topic messages must adhere to the following format:

```
{
  "p": "hcs-8",
  "op": *string*,
  "sid": *array*,
  "d": *string*,
  "m": *string*"
}
```

Where:

| Field | Description                                                                                                                                           |
|-------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| p     | Protocol used by the poll. Polls that follow this framework must use: hcs-8  |
| op    | Operation being executed |
| sid   | Sequence information for an operation with data that spans multiple HCS messages. (Register only) |
| d     | Operation-specific JSON-formatted data. Following HCS-1 conventions, the data should be prefixed by its type. For this HIP, every message should be: data:application/json;utf8,\{JSON_DATA\} |
| m     | Memo |

### Multi-Message Operation - Register operation only

HCS messages have a length limit of 1024, so the 'sid' field (standing for 'Sequence Id') is introduced to allow for operations to be posted with arbitrary length.

**Only the register operation is permitted to be a multi-message operation**, for the following reasons:

- The register operation is the only complex operation with arbitrary fields that could greatly increase the size of the message.
- Voting, Managing and Updating are all simple operations that don't require much metadata, so the need for long messages is currently seen as not important to support.
- Multi-message operations significantly increase the complexity of parsing the topic, since messages are not guaranteed to be in order (without adding in extra safeguards) and are not guaranteed to be concurrent (other users can post in between the messages by chance).

If the operation fits into a single HCS message, then sid should be omitted.

The format of sid is a three-element number array consisting of a unique id (uid), a sequence number (num) and a sequence length (len):

`"sid" : [ uid, num, len ]`

The JSON Schema for sid is here: https://www.rockvt.com/schemas/hcs-yy-schema/sequence-schema/v1.0.0.json

- **UniqueID (uid)**: The identifier for this operation. This identifier is an integer and must be unique to identify all messages that share the same operation. The number "0" is reserved for the Register operation.
- **Sequence Number (num)**: The index of the message in the array, starting at 0.
- **Length (len)**: The length of the array.

#### Parsing rules:

- All messages with a particular UniqueId (uid) must be submitted by the same account.
- uid's are sequential over the whole topic. This puts the requirement on the message sender to find the most recent uid in the topic and increment it for their own message.
- If two accounts submit messages with the same uid, the account that submitted the first message that contains the uid by consensus timestamp is considered valid, and all other messages from other accounts with that uid are invalid.
- The number of messages with a UniqueID must equal the given Length. Sequences with missing messages or more messages than Length are discarded as a whole (For example, if one or more messages were not submitted to the topic).
- Length must be constant for all messages associated with a UniqueId.
- "op" must be constant for all messages associated with a UniqueId.
- Only the memo from the first message (num) should be set. The rest of the memos should not be set and should be discarded.
- The timestamp of the latest message in the array is the timestamp used for evaluating the operation.

The "d" field of all the messages should be directly concatenated to form the complete metadata string. The operation is then evaluated normally.

An example two-message operation is shown below. For clarity's sake, the data is shorter than the 1024 requirement. A message this short would not need to be split into multiple messages in practice.


Message 1:
```
data:application/json;utf8,
{
  “p”:”hcs-8”,
  “op”:”register”,
  “sid”: [0,0,2],
  “d”: “{
        “title”: “An incomplete poll”, 
    “,
   “m”: “hcs-8”
}
```

Message 2:
```
data:application/json;utf8,
{
  “p”:”hcs-8”,
  “op”:”register”,
  “sid”: [0,1,2],
  “d”: “
        “author” : “A. Pollmaker”
    } 
    “
}
```

Data after concatenation of Message 1 & 2 and removing data type declaration:
```
“{
        “title”: “An incomplete poll”, 
        “author”: “A. Pollmaker”
 }”
```

Final operation from the perspective of the application:
```

{
  “p”:”hcs-8”,
  “op”:”register”,
  “d”:
“{
        “title”: “An incomplete poll”, 
        “author”: “A. Pollmaker”
}”,
   “m”:”poll memo.”
}
```

