---
title: Token Data Implementation
---

#### [ hcs-9 - Poll Metadata Schema ]

Token data objects are used in the definition of a token gate to describe the token(s) that are used in the gate.

```
{
    "tokenId": "0x1234567890abcdef",
    "name": "Descriptive Name for voters",
    "description": "Description for voters",
    "uri": "uri to token data - optional",
    "serials": "comma-separated list of serial numbers or serial number ranges - optional",
    "tokensPerVote": 1,
    "multiplier": 1
}
```

### Token Data Schema

#### Serials

The serials field is a comma-separated list of serial numbers or serial number ranges.

Example: "1,2,3,4-6,7-9"

