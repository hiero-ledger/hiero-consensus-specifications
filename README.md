# Hiero Consensus Specifications

| ![](./Hashgraph-Online.png) | Canonical, text-first specifications for **Hiero Consensus Standards (HCS)**, originally created and maintained by [Hashgraph Online](https://hol.org).<br><br>Reference implementations: [Standards SDK](https://hol.org/docs/libraries/standards-sdk/overview)<br><br>Process and governance are defined in `docs/standards/hcs-4.md`. |
| :-- | :-- |

## Abstract

Hiero Consensus Standards (HCS) are open, versioned specifications for interoperable data formats and protocol patterns layered on top of the Hiero Consensus Service (and similar consensus/message transports). They support patterns ranging from simple file storage and registry management to complex state and discovery workflows for appnets and AI agents. This repository is the canonical source for the normative text of HCS specifications, along with the contribution workflow and governance process used to evolve them.

## Motivation

Implementers (apps, wallets, agents, indexers, and tooling) need a stable, auditable source of truth for message formats, validation rules, and lifecycle expectations so independent implementations can interoperate reliably. A lightweight, specifications-first repository makes review and versioning easier, reduces ambiguity, and keeps reference implementations (like the Hashgraph Online Standards SDK) aligned.

## Specifications

| Spec | Title | What it does |
| --- | --- | --- |
| [HCS-1](docs/standards/hcs-1.md) | HCS-1: File Management | Encode, chunk, upload, retrieve, and reassemble file data on HCS topics using a JSON schema and topic memo validation. |
| [HCS-2](docs/standards/hcs-2.md) | HCS-2: Topic Registries | Defines topic registries for HCS, enabling structured data organization, discovery, and retrieval. |
| [HCS-3](docs/standards/hcs-3.md) | HCS-3: Recursion | Defines a method for implementing recursion on HCS, enabling more complex data structures and relationships. |
| [HCS-4](docs/standards/hcs-4.md) | HCS-4 — HCS Standardization Process | Defines the lifecycle, roles, criteria, and repository workflow for proposing, reviewing, approving, publishing, and maintaining Hiero Consensus Standards (HCS). |
| [HCS-5](docs/standards/hcs-5.md) | HCS-5: Hashinals | Standard way to inscribe Hashinals using Hiero Consensus Service and Hiero Token Service, inspired by Ordinals. |
| [HCS-6](docs/standards/hcs-6.md) | HCS-6: Dynamic Hashinals | Framework for creating dynamic Hashinals whose content/behavior can be updated over time. |
| [HCS-7](docs/standards/hcs-7.md) | HCS-7: Smart Hashinals | Enables dynamic NFTs whose metadata updates based on smart contract state using deterministic topic selection and WASM processing. |
| [HCS-8](docs/standards/hcs-8/index.md) | HCS-8 Standard: Poll Topic Standard | Framework for using HCS topics to manage polls and record poll activity on-ledger. |
| [HCS-9](docs/standards/hcs-9/index.md) | HCS-9: Poll Metadata | Metadata schema for polls intended for broad ecosystem adoption and interoperability. |
| [HCS-10](docs/standards/hcs-10/index.md) | HCS-10 OpenConvAI Standard: AI Agent Communication on HCS | Framework for AI agents to register, discover, and communicate over HCS using registries, topics, and standardized operations. |
| [HCS-11](docs/standards/hcs-11.md) | HCS-11: Profile Metadata | Systematic approach for managing profiles on Hiero via account memos, enabling rich identity for individuals and AI agents. |
| [HCS-12](docs/standards/hcs-12/index.md) | HashLinks — Actions, Blocks, and Assembly | Framework for building interactive experiences on Hiero using WASM actions, UI blocks, and an assembly layer without smart contracts. |
| [HCS-13](docs/standards/hcs-13.md) | HCS-13: Schema Registry | Standard way to define, register, and reference schemas for validation and interoperability across the Hiero ecosystem. |
| [HCS-14](docs/standards/hcs-14.md) | HCS-14 - Universal Agent ID Standard | Systematic approach for generating globally unique identifiers for AI agents across web2 and web3 environments. |
| [HCS-15](docs/standards/hcs-15.md) | HCS-15 Standard: Petals - Profile Accounts | Enables account holders to create multiple account instances (petals) that share the same private key while isolating profiles and assets. |
| [HCS-16](docs/standards/hcs-16.md) | HCS-16 Standard: Floras - AppNet Accounts | Rules and practices for multi-party Flora accounts enabling coordination, shared escrow, and state consensus across agents. |
| [HCS-17](docs/standards/hcs-17.md) | HCS-17 Standard: State Hash Calculation | Methodology for calculating state hashes of accounts and formations to support auditable, tamper-proof state verification. |
| [HCS-18](docs/standards/hcs-18.md) | HCS-18 Standard: Flora Discovery Protocol | Discovery and formation protocol for Floras via an open broadcast mechanism. |
| [HCS-19](docs/standards/hcs-19.md) | HCS‑19 Standard: AI Agent Privacy Compliance | ISO/IEC TS 27560-aligned framework for AI agents to document and manage privacy compliance on Hiero via HCS topics (consent, processing records, rights fulfilment). |
| [HCS-20](docs/standards/hcs-20/index.md) | HCS-20 Standard: Auditable Points Standard on Hiero | Defines how points can be managed and audited on Hiero using HCS topics, inspired by BRC-20 and extended for auditable points and inscriptions. |
| [HCS-21](docs/standards/hcs-21.md) | HCS-21 Standard: Adapter Registry | Platform-agnostic adapter registry packaging decentralized adapters, manifests, and consensus context for distributed appnets (Floras). |
| [HCS-XX](docs/standards/hcs-XX.md) | HCS-XX — Template | Template for drafting a new specification that conforms to HCS-4 requirements. |

Full index: `docs/standards/index.mdx`

To propose a new specification, follow the [HCS-4 process](docs/standards/hcs-4.md#submission-and-review-workflow).

## Contribution Process

- Canonical process: [HCS-4 — HCS Standardization Process](docs/standards/hcs-4.md)
- New proposals: open an issue (use “01 - Proposal”), draft under `docs/standards/drafts/<slug>.md` (start from `docs/standards/hcs-XX.md`), then request number assignment and submit a PR
- Updates to existing specs: submit a PR against the relevant `docs/standards/hcs-*` document(s) following the HCS-4 change management rules
- Normative keywords: use RFC 2119 language (MUST/SHOULD/MAY) where appropriate

## Implementations

Reference implementations live in the Hashgraph Online Standards SDK:

- https://hol.org/docs/libraries/standards-sdk/overview

## Governance

Editors and maintainers are listed in `MAINTAINERS.md`. CODEOWNERS are defined in `CODEOWNERS`.
