---
title: Rejected Ideas
sidebar_position: 4
---

#### [ hcs-8 - Poll Topic Standard ]

This section serves as a record of ideas which were rejected for inclusion into the final draft, alongside the reasoning for the rejection.

### Future Discussion Point: Topic Memo format

This version of hcs-8 requires the topic creation transaction memo to be exactly 'hcs-8'. This leaves open the ability to extend the standard to use the memo more in the future. 

For example, if the schema for the topic could be located in another topic, more information could be added as follows:

hcs-8:0 = default (find schema on message 1)

hcs-8:1:0.0.12345 (find schema on 0.0.12345)

in this case, 'hcs-8' would be equivalent to 'hcs-8:0'

For now such information is not required.

### Using the built in HCS message sequence ID for the multi-message operations

It is somewhat elegant to use the sequence ID of a message when posting multi-message operations which serves as a unique identifier for a message. However this adds a lot more logic and time to post large files. Messages must now be posted in sequence, and the receipt for the previous message must be received before the next message is posted. This significantly reduces the practicality of this idea.

### Using the user’s account ID plus a suffix to create a uid for multi-message operations

Original concept:

The uid should be a user’s account number, without the “0.0.” prefix, example: user 0.0.209838’s first multi-message operation should use uid “209838”.
If a user posts multiple multi-message operations, append the subsequent messages with “-” and an incrementing number. example: “209838-1”

However Michael Kantor raises a good point: “To make it easier for indexers, I wonder if there's a way we can avoid the `-` in the operation? I am thinking so its easier to call a `sort` operation for example.”
