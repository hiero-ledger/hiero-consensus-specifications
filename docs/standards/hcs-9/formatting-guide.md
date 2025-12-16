---
title: Formatting Guide
sidebar_position: 6
---

**Schema**

Schema is a required field that defines the schema of the module. All modules that follow the base schema use hcs-9 as the schema, but developers may define their own schema for their own modules. Developers should parse the schema to determine the type of module and how to process the data, and display appropriate information when the schema is not recognized.

**UUID**

UUID is a unique identifier for modules for use within the poll metadata. This is a generated value which should be unique within a single poll.

**CSV format**

CSV format is a comma separated list of values. It is used to define lists of values for fields that have a limited set of possible values. Standard CSV formatting should be used:

See the example below, where 0.0.55555 represents and NFT and 0.0.66666 represents an FT. This particular snapshot includes data for both those tokens in the CSV.

```
#snapshotDate: [UNIX Timestamp]
accountId,tokenId,balance,serials
"0.0.12345","0.0.55555",2,"1,12"
"0.0.67890","0.0.66666",5
"0.0.12345","0.0.66666",25
```
**URI Formatting**

URI’s shall follow the following format: protocol://resource_location

For resources that are on the world wide web, the standard https protocol is acceptable. Ie. http://www.example.org/image/file.jpg

For resources that are on IPFS, the protocol must be ipfs:// and the resource location must be the cid of the file. Ie. ipfs://bafkreibwci24bt2xtqi23g35gfx63wj555u77lwl2t55ajbfjqomgefxce

For resources that are on Arweave, the protocol must be ar:// and the resource location must be the cid of the file. Ie. ar://bafkreibwci24bt2xtqi23g35gfx63wj555u77lwl2t55ajbfjqomgefxce

The onus is placed on dApps to take the cid and access the file information through the method of their choosing.

CDN links such as Cloudflare and Infura are not acceptable. These are not primary sources. Ie. https://cloudflare-ipfs.com/ipfs/bafkreibwci24bt2xtqi23g35gfx63wj555u77lwl2t55ajbfjqomgefxce

IPFS CIDS may contain file paths or extensions as long as they adhere to best practices as described here: https://docs.ipfs.io/how-to/best-practices-for-nft-data/#persistence-and-availability

For resources that are on the hedera file service, the protocol is hedera://

A more complete list of URI’s can be found here: https://en.wikipedia.org/wiki/List_of_URI_schemes