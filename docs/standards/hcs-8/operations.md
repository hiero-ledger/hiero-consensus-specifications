---
title: Operations
sidebar_position: 2
---

#### [ hcs-8 - Poll Topic Standard ]

The following operations are defined:

| Name        | Operation string (case sensitive) |
|-------------|-----------------------------------|
| Register    | register                          |
| Manage      | manage                            |
| Vote        | vote                              |
| Update      | update                            |
| Information | information                       |

---

### Register

The Register operation must be the first message in the topic. Topics without a valid register operation as the first message are not considered valid hcs-8 polls. 

The metadata for the register operation should be a JSON-formatted string. Poll metadata is recommended to follow hcs-9: Poll Metadata Standard to allow for cross-platform compatibility.

When the Register metadata exceeds the maximum length of a HCS message (1024 characters), follow up Register operations are posted, as many as needed to complete the JSON string.

Only the first message in the topic, or the first set of messages (as identified by the ‘sid’ field) are legal register operations. Any register operations that occur after this are invalid and ignored.

**Sample Message:**

```
data:application/json;utf8,{
  “p”:”hcs-8”,
  “o”:”register”,
  “d”: “{
   "schema":"”hcs-9”",
   "title":"”What is your favourite colour?”",
   "description":"A sample poll with one vote per voter.",
   "author":"0.0.1234567",
   "voting-rules":{
      "schema":"hcs-9",
      "ruleset":"single-vote"
   },
   "update-rules":{
      "schema":"hcs-9",
      "update-settings":{
         "endDate":true
      }
   },
   "options":{
      "schema":"hcs-9",
      "items":[
         {
            "id":0,
            "title":"red",
            "description":"The colour of fire."
         },
         {
            "id":1,
            "title":"blue",
            "description":"The colour of water."
         },
         {
            "id":2,
            "title":"green",
            "description":"The colour of earth."
         }
      ]
   },
   "status":"inactive",
   "startDate":"1720627200",
   "endDate":"1723305600"
}”,
  “m”:”0.0.1234567 registers poll with initial status 'inactive' and title 'What is your favourite colour?'”
}
```

**Register operation memo convention**

The memo should be written to convey the operation in clear and human-readable terms. It is recommended that the memo contain the author, 'register' action, initial status and title of the poll.

---

### Manage

The Manage operation manages the state of the poll. 

The data contains the following fields:

accountId - The account calling the manage operation
action - The action being called (case sensitive)


| Action | Description                                                                                                                                                                                                                                                            |
|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| open   | Manually opens the poll for voting. Votes that occur after this message are counted toward the total vote. Note that this is not the only way to open votes. For example, if startDate is set during registration, the poll implicitly opens after the specified date. |
| close  | Closes the poll for voting. Votes that occur after this message are NOT counted toward the total vote. This is a permanent status change. The poll cannot be opened after Close is called. For temporary closing the poll, use pause. |
| cancel | Closes the poll for voting, but declares the poll cancelled. Provide additional details in the memo. This is a permanent status change. The poll cannot be opened after Cancel is called. For temporary closing the poll, use pause. |
| pause  | Closes the poll for voting temporarily. Provide additional details in the memo.  |

On a technical level there is no difference between a 'close' and 'cancel' action. However, semantically it is understood that the result of a closed poll is valid, while the result of a cancelled poll is invalid.

**Sample message:**

``` 
data:application/json;utf8,{
  “p”:”hcs-8”,
  “op”:”manage”,
  “d”:”{ 
        “accountId”: “0.0.12345”,
        “action”: “cancel”
        }”
  “m”:”Poll cancelled due to community feedback. We will reopen it shortly with updated options!”
}
```

** Manage operation memo convention**

The memo should be written to convey the reason for the operation in a conversational manner as a manager of the poll.

---

### Vote

The Vote operation registers a vote action. The format of the voting metadata is determined by the voting rules which are defined in the Register metadata.

**Voter Signature is Required:** The user placing the vote must sign the HCS Transaction in order for the operation to be considered valid.

The most basic Vote action is to cast a vote for an option. However other vote actions could be casting multiple votes for options, or withdrawing votes that the user has previously cast.

The format of the metadata for the vote action is defined in the metadata standard being used.

**Sample message:**

```
data:application/json;utf8,{
  “p”:”hcs-8”,
  “op”:”vote”,
  “d”:[
     {
        “optionId”:”0”,
        “weight”:”1”
     }
   ]
   “m”:”0.0.123456 casts 1 vote for option 0: Red.”
}
```

**Vote operation memo convention**

The memo is required to be a human-readable statement of the vote action, written in active tense. Important data to convey: The user’s name/identifier, the action (“Cast a vote”), the weight of the vote, and the option being selected (the title and the id). This is to maximize readability of the topic.

---

### Update

The Update operation changes the settings of the poll, per the rules set in the initial settings.
 
Similar to Vote, the available legal actions depend on the Register metadata. If the register metadata does not explicitly grant permission for the author to change the poll options, then an update operation to change a poll option would be illegal.

The contents for the metadata field for the Update action is defined in the metadata standard being used.

**Sample message:**

```
data:application/json;utf8,{
  “p”:”hcs-8”,
  “op”:”update”,
  “md”:
      {
         “change”: {
            “endDate”:”1689620400”
         }
      },
  “m”:”0.0.246810 updated the poll: endDate changed to July 17, 2024 at 12:00PM PST.”
}

```

**Update operation memo convention**

The memo is required to be a human-readable statement of the update action, written in active tense. This should be generated based on the data rather than written by the updater and contain the user ID, the operation 'update', and a summary of the changes. 

---

### Information

The Information operation is a way to post messages for poll platforms to display. It is up to the poll creator to decide how it is used.

Metadata is optional but could be extended to provide more functionality.

**Sample message:**

```
data:application/json;utf8,{
  “p”:”hcs-8”,
  “op”:”information”,
  “m”:”The final tally of votes is: Red 21, Blue 15 and Green 5. Red is the winning option.”
}
```

**Information operation memo convention**

The memo should be written to convey information in a conversational manner.