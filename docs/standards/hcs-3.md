---
title: "HCS-3: Recursion"
description: The HCS-3 standard defines a method for implementing recursion within Hedera Consensus Service, enabling more complex data structures and relationships.
sidebar_position: 3
---

# HCS-3 Standard: Recursion within Hedera Consensus Service

## Status: Published

## Table of Contentss

- [HCS-3 Standard: Recursion within Hedera Consensus Service](#hcs-3-standard-recursion-within-hedera-consensus-service)
  - [Status: Published](#status-published)
  - [Table of Contentss](#table-of-contentss)
  - [Abstract](#abstract)
  - [Motivation](#motivation)
  - [Specification](#specification)
    - [Protocol Overview](#protocol-overview)
    - [Resource Referencing](#resource-referencing)
    - [CDN Integration](#cdn-integration)
  - [Implementation Guidelines](#implementation-guidelines)
    - [Resource Loading](#resource-loading)
    - [Error Handling](#error-handling)
    - [Caching](#caching)
    - [Configuration Variables](#configuration-variables)
    - [Data Attributes](#data-attributes)
  - [Examples](#examples)
    - [JavaScript](#javascript)
    - [CSS](#css)
    - [Image](#image)
    - [WebAssembly Module](#webassembly-module)
    - [JSON Data](#json-data)
    - [Audio](#audio)
    - [Video](#video)
  - [Security Considerations](#security-considerations)
  - [Future Work](#future-work)
  - [Conclusion](#conclusion)

## Abstract

This document introduces HCS-3, a protocol extension for the Hedera Consensus Service (HCS), enabling recursive referencing of inscriptions within the Hedera Hashgraph. HCS-3 aims to standardize the method of referencing and loading resources stored as HCS messages, facilitating the creation of complex, interconnected decentralized applications.

## Motivation

The primary goal of HCS-3 is to leverage Hedera's high-throughput, low-latency consensus mechanism to enable complex data structures and references within the Hedera Hashgraph. By standardizing resource referencing and loading, HCS-3 seeks to foster a new level of integration for decentralized applications built on Hedera.

## Specification

### Protocol Overview

HCS-3 extends HCS by introducing a standardized way to reference on-graph files inside of applications.

### Resource Referencing

Resources are referenced using the following [HRL](../definitions.md#hashgraph-resource-locator) structure:

```
hcs://{protocol_number}/{topic_id}
```

- `protocol_number`: Identifies the specific HCS protocol version.
- `topic_id`: The Hedera topic ID where the resource is stored.

### CDN Integration

To facilitate efficient resource loading, HCS-3 supports integration with Content Delivery Networks (CDNs). CDNs typically do the heavy lifting of retrieving all messages for an HCS-1 file and combining them for usage. The standard CDN URL format is:

```
https://{cdn_domain}/api/hashinals-cdn/{topic_id}?network={network}
```

- `cdn_domain`: The domain of the CDN service.
- `topic_id`: The Hedera topic ID of the resource.
- `network`: The Hedera Hashgraph (e.g., 'mainnet', 'testnet').

Example using the TierBot CDN:

```
https://kiloscribe.com/api/hashinals-cdn/0.0.1234567?network=mainnet
```

## Implementation Guidelines

### Resource Loading

1. Parse [HRLs](../definitions.md#hashgraph-resource-locator) inside of HTML elements to extract the topic ID and protocol number.
2. Use the extracted information to construct the appropriate CDN URL or Mirror Node API URL.
3. Fetch the resource from the CDN or Mirror Node API.
4. Process and render the resource based on its type (e.g., script, stylesheet, image).

### Error Handling

1. Implement retry logic for failed resource loads.
2. Provide fallback mechanisms for unavailable resources.
3. Log and report errors to facilitate debugging and monitoring.

### Caching

1. Implement client-side caching to improve performance and reduce network requests.
2. Respect cache control headers provided by the CDN.
3. Implement a cache invalidation strategy to ensure resource freshness.

### Configuration Variables

The following configuration variables can be detected within a script configuration file. An implementation client should seek to use these variables to provide flexibility for end-users.

| **Configuration Variable** | **Description**                                                          | **Default Value**                           |
| -------------------------- | ------------------------------------------------------------------------ | ------------------------------------------- |
| `cdnUrl`                   | The base URL for the CDN where resources are loaded.                     | `https://kiloscribe.com/api/hashinals-cdn/` |
| `network`                  | Specifies the Hedera Hashgraph, such as `mainnet` or `testnet`.          | `mainnet`                                   |
| `retryAttempts`            | Number of retry attempts for failed resource fetches.                    | `3`                                         |
| `retryBackoff`             | Time in milliseconds between retry attempts, with exponential backoff.   | `300`                                       |
| `debug`                    | Toggles debug logging.                                                   | `false`                                     |
| `showLoadingIndicator`     | If set to true, displays loading status in the console.                  | `false`                                     |
| `loadingCallbackName`      | The name of the global function to call when the loading status changes. | `null`                                      |

Example script:

```html
<script
  data-hcs-config
  data-hcs-cdn-url="https://kiloscribe.com/api/hashinals-cdn/"
  data-hcs-network="mainnet"
  data-hcs-debug="true"
  data-hcs-retry-attempts="5"
  data-hcs-retry-backoff="500"
  data-hcs-show-loading-indicator="true"
  data-hcs-loading-callback-name="setLoadingIndicator"
></script>
```

### Data Attributes

To ensure proper loading and execution of resources, the following data attributes are used in HTML elements.

| **Attribute**     | **Description**                                                                  | **Default Value**                         |
| ----------------- | -------------------------------------------------------------------------------- | ----------------------------------------- |
| `data-src`        | Specifies the HCS URL to load the resource from.                                 | `null`                                    |
| `data-script-id`  | A unique identifier for the script or resource being loaded.                     | `null`                                    |
| `data-required`   | Marks the resource as required, causing the process to halt if it fails to load. | `false`                                   |
| `data-load-order` | Specifies the load order for resources, ensuring proper sequence.                | `undefined` (loads last if not specified) |
| `data-network`    | Overrides the default network configuration for specific resources.              | `mainnet`                                 |

## Examples

Below are detailed examples of how to reference and use various resource types using the HCS-3 standard:

### JavaScript

```html
<script
  data-src="hcs://1/0.0.6614307"
  data-script-id="threejs"
  data-required="true"
  data-load-order="1"
  data-network="mainnet"
></script>
```

This script tag loads a JavaScript file (Three.js) from the specified HCS topic. It's marked as required and set to load first.

### CSS

```html
<link
  rel="stylesheet"
  data-src="hcs://1/0.0.6633438"
  data-script-id="nes-css"
  data-load-order="2"
/>
```

This link tag loads a CSS file (NES.css) from the specified HCS topic, set to load second.

### Image

```html
<img data-src="hcs://1/0.0.6529019" alt="Hedera Logo" data-load-order="3" />
```

This img tag loads an image from the specified HCS topic, set to load third.

### WebAssembly Module

```html
<script
  data-src="hcs://1/0.0.6628687"
  type="application/wasm"
  data-script-id="rust-wasm"
  data-load-order="4"
></script>
```

This script tag loads a WebAssembly module from the specified HCS topic, set to load fourth.

### JSON Data

```html
<script
  data-src="hcs://1/0.0.6650001"
  type="application/json"
  data-script-id="config-data"
  data-load-order="5"
></script>
```

This script tag loads a JSON data file from the specified HCS topic, set to load fifth.

### Audio

```html
<audio data-src="hcs://1/0.0.6660001" data-load-order="6" controls></audio>
```

This audio tag loads an audio file from the specified HCS topic, set to load sixth.

### Video

```html
<video
  data-src="hcs://1/0.0.6670001"
  data-load-order="7"
  controls
  width="640"
  height="360"
></video>
```

This video tag loads a video file from the specified HCS topic, set to load seventh.

## Security Considerations

1. Implement content integrity checks to ensure loaded resources haven't been tampered with.
2. Use HTTPS for all CDN connections to prevent man-in-the-middle attacks.
3. Implement strict CORS policies to prevent unauthorized resource access.
4. Regularly audit referenced resources for potential security vulnerabilities.

## Future Work

1. Explore integration with other Hedera services for enhanced functionality.
2. Develop standardized tools and libraries to simplify HCS-3 implementation.
3. Investigate cross-chain resource referencing capabilities.
4. Enhance support for complex data structures and resource dependencies.

## Conclusion

HCS-3 provides a standardized method for referencing and loading resources within the Hedera Hashgraph, enabling the creation of more complex and interconnected decentralized applications. By leveraging CDN integration and providing clear implementation guidelines, HCS-3 ensures efficient and reliable access to referenced inscriptions, fostering a new ecosystem of dynamic, on-graph applications on the Hedera Hashgraph.
