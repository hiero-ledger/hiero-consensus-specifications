---
title: "HCS-1: File Management"
description: The HCS-1 standard provides a systematic approach to encode, chunk, upload, retrieve, and reassemble file data for applications using Hedera Consensus Service (HCS). This process is agnostic of the implementation details, focusing on the JSON structure and the use of a Topic ID for efficient data management.
sidebar_position: 1
---

# HCS-1 Standard: File Data Management with Hedera Consensus Service

### Status: Published

### Table of Contents

- [HCS-1 Standard: File Data Management with Hedera Consensus Service](#hcs-1-standard-file-data-management-with-hedera-consensus-service)
    - [Status: Published](#status-published)
    - [Table of Contents](#table-of-contents)
  - [Authors](#authors)
  - [Abstract](#abstract)
  - [Motivation](#motivation)
  - [HCS Topic Validation](#hcs-topic-validation)
    - [Memo Structure](#memo-structure)
    - [Supported Compression Algorithms](#supported-compression-algorithms)
    - [Supported Encodings](#supported-encodings)
  - [Encoding and Chunking](#encoding-and-chunking)
    - [Uploading to HCS](#uploading-to-hcs)
  - [Retrieving and Reassembling](#retrieving-and-reassembling)
  - [Conclusion](#conclusion)

## Authors

- Kantorcodes [https://twitter.com/kantorcodes](https://twitter.com/kantorcodes)

### Additional Authors
- Patches [https://twitter.com/tmcc_patches](https://twitter.com/tmcc_patches)

## Abstract

The HCS-1 standard provides a systematic approach to encode, chunk, upload, retrieve, and reassemble file data for applications using Hedera Consensus Service (HCS). This process is agnostic of the implementation details, focusing on the JSON structure and the use of a Topic ID for efficient data management.

## Motivation

Storing data on-chain has become increasingly important for verification, immutability, and scale of web3 applications. This specification will enable future use cases like **Hashinals**, truly _decentralized_ applications, and enhanced peer-to-peer communication without the need of typical cloud infrastructure.

## HCS Topic Validation

HCS-1 topics must

- include a SHA-256 hash of the file being uploaded, the compression algorithm being used, and the encoding as the **memo**. Viewers of HCS-1 data will read the memo to understand if the file data is valid by comparing the file hash to the memo. Please read [Memo Structure](#memo-structure) for details on how to properly generate a memo.
- include a Submit Key. HCS-1 Topics without a Submit Key will automatically be marked as invalid files, and will be ignored.
- NOT include an Admin Key. HCS-1 Topics with an Admin Key will automatically be marked as invalid files, and will be ignored. This ensures that data cannot be deleted, reducing risk for all participants in the protocol.
  In summary, a valid HCS-1 Topic includes a SHA-256 hash of the file being uploaded in its memo, a submit key, and no admin key.

### Memo Structure

The format for the memo is as follows:

`[hash]:[algo]:[encoding]`

- [hash] is the SHA-256 hash of the file being uploaded before any compression
- [algo] is the compression algorithm being used, for example, `zstd` or `brotli`
- [encoding] is the encoding used to store the compressed file, for example, `base64`

For example a valid memo would like:

`532eaabd9574880dbf76b9b8cc00832c20a6ec113d682299550d7a6e0f345e25:zstd:base64`

### Supported Compression Algorithms

| Algorithm | Description |
|-----------|-------------|
| zstd | Recommended for most media types like images, videos, and audio |
| brotli | Recommended for most text-based formats like JSON, HTML, and CSS |

### Supported Encodings

| Encoding | Description |
|-----------|-------------|
| base64 | Standard base64 encoding |

If you'd like a different compression algorithm or encoding to be approved, please raise a pull request.

## Encoding and Chunking

Before uploading to HCS, file data must be formatted correctly by following these three steps

1. The entire file should be compressed using zstd, this ensures cost-savings on all ends of the protocol in a lossless format.

- Plenty of implementations for zSTD exist, for example, using NodeJS you could use the MongoDB library: https://github.com/mongodb-js/zstd/tree/main
- `const compressedFile = await compress(file, 10); `
- We recommend using a compression level of 10, but any compression level should work.

2. After compressing, the resulting data (typically a buffer) should be converted into a base64 string
3. The final base64 string should chunked into segments no greater than 1024 bytes. Each chunk is encapsulated in a JSON object with two attributes:

- `o`: The order index indicating the chunk's sequence in the overall file.
- `c`: The chunk's content, a substring of the base64-encoded file. The first chunk aka, `o = 0` should include a data prefix for the mime type.

The format for the data prefix is
`data:[mimeType];base64`
For example:
`data:image/png;base64,`

The format for each chunk in a message is as follows:

```json
{
  "o": 0,
  "c": "base64-encoded-chunk"
}
```

### Uploading to HCS

Each chunk is uploaded to a Hedera Consensus Service topic as an HCS message. Keep in mind that because of the `o` property in the JSON schema, the sequence number that the chunk is uploaded in does not matter. This enables uploading many chunks in parallel.

## Retrieving and Reassembling

To display the data, the application retrieves all chunk messages from the specified HCS topic. Chunks are sorted by their order index (`o`) and concatenated based on their content (`c`). The combined base64 string is then decoded back into binary data for display.

The complete steps are:

1. Fetch all messages from the Topic ID, and sort them by their index `o`
2. Concatenate the messages together, and decompress the file using a `zstd` decoding library.
3. Ensure the memo of the Topic ID matches the SHA-256 hash of the decompressed file data.
4. Convert the uncompressed base64 string into binary data

## Conclusion

The HCS-1 standard outlines a robust protocol for managing file data sets of all sizes and mime types within the Hedera Consensus Service, leveraging JSON for data structuring and a registry for efficient topic management. This approach ensures data integrity, facilitates easy data retrieval, and supports scalable application development on Hedera.
