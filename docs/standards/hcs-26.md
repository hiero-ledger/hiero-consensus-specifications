---
title: "HCS-26: Decentralized Agent Skills Registry"
description: "Standard for registering versioned agent skills on HCS using HCS-2 registries and HCS-1 manifests."
sidebar_position: 26
---

# HCS-26 Standard: Decentralized Agent Skills Registry

### Status: Draft

### Version: 1.0

### Table of Contents

- [Authors](#authors)
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Specification](#specification)
- [Rationale](#rationale)
- [Backwards Compatibility](#backwards-compatibility)
- [Security Considerations](#security-considerations)
- [Privacy Considerations](#privacy-considerations)
- [Test Vectors](#test-vectors)
- [Conformance](#conformance)
- [References](#references)
- [Governance Record (fill at publication)](#governance-record-fill-at-publication)
- [License](#license)

## Authors

- Jake Hall (https://x.com/jaycoolh)

## Abstract

HCS-26 defines a decentralized registry for Agent Skills using HCS-2 topic registries with versioning and HCS-1 file storage. Each skill is identified by the sequence number of its discovery registration message. Skill versions reference a manifest stored on HCS-1, which maps a logical folder of skill files and optional subdirectories such as `scripts/`, `references/`, and `assets/`.

## Motivation

Agent Skills are increasingly shared across ecosystems and need a verifiable, decentralized way to publish, discover, and version skill packages. Existing standards provide file storage (HCS-1) and registries (HCS-2), but do not define a cohesive workflow for skill publication, version management, and structured skill folders. HCS-26 fills this gap with a minimal, composable design that works with existing HCS infrastructure.

## Specification

Normative language in this document uses the key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY as described in RFC 2119.

### Architecture Overview

HCS-26 uses two HCS-2 registries:

- **Skill Discovery Registry**: a global, indexed HCS-2 topic that records skills. The sequence number of the `register` message becomes the skill identifier (`skill_uid`).
- **Skill Version Registry**: an indexed HCS-2 topic per skill, listing versions and pointing to the HCS-1 manifest (`SKILL.json`).

A skill is a logical folder described by a manifest and a set of HCS-1 files. The manifest maps file paths to HCS-1 HRLs and includes metadata.

### Terminology

- **Skill UID**: the sequence number of the discovery registry `register` message.
- **Discovery Registry**: the HCS-2 registry topic where skills are registered.
- **Version Registry**: the HCS-2 registry topic that lists versions of a single skill.
- **Manifest**: the `SKILL.json` file stored on HCS-1 describing the skill folder.
- **HRL**: Hashgraph Resource Locator `hcs://{protocol}/{topic_id}`.

Implementations MUST treat a skill as the tuple `(discovery_registry_topic_id, skill_uid)` to ensure global uniqueness.

### Topic Types and Memo Format

HCS-26 uses the HCS-2 memo convention with the following format:

```
hcs-26:{indexed}:{ttl}:{type}
```

**Type Enum Values**

| Type Enum | Topic Type              | Description                          |
| --------- | ----------------------- | ------------------------------------ |
| `0`       | Skill Discovery         | Global discovery registry (indexed) |
| `1`       | Skill Version Registry  | Per-skill version registry (indexed) |
| `2`       | Reputation Topic        | Optional reputation registry         |

**Recommended memos**

- Discovery registry: `hcs-26:0:86400:0`
- Version registry: `hcs-26:0:86400:1`
- Reputation topic: `hcs-26:0:86400:2`

### Transaction Memo Format (Analytics)

Every HCS-26 operation SHOULD include a transaction memo for analytics:

```
hcs-26:op:<operationEnum>:<topicTypeEnum>
```

| Operation        | Enum |
| ---------------- | ---- |
| `register`       | `0`  |
| `update`         | `1`  |
| `delete`         | `2`  |
| `migrate`        | `3`  |

| Topic Type | Enum |
| ---------- | ---- |
| discovery  | `0`  |
| version    | `1`  |
| reputation | `2`  |

### Discovery Registry Operations

Discovery registry messages extend HCS-2 and use `p: "hcs-26"`.

#### Register (Skill Creation)

```json
{
  "p": "hcs-26",
  "op": "register",
  "version_registry": "0.0.123456",
  "publisher": "0.0.78910",
  "metadata": {
    "name": "PDF Processing",
    "description": "Extract and clean PDF text",
    "author": "Example Labs",
    "license": "Apache-2.0",
    "tags": ["pdf", "text", "extraction"],
    "languages": ["python"],
    "homepage": "https://example.com/skills/pdf-processing"
  },
  "m": "optional memo"
}
```

| Field              | Type        | Required | Description                                                     |
| ------------------ | ----------- | -------- | --------------------------------------------------------------- |
| `p`                | string      | Yes      | Protocol identifier (`hcs-26`).                                 |
| `op`               | string      | Yes      | `register`.                                                     |
| `version_registry` | string      | Yes      | Topic ID for the per-skill version registry.                    |
| `publisher`        | string      | Yes      | Publisher account ID or UAID.                                   |
| `metadata`         | object      | Yes      | Skill metadata (schema below).                                  |
| `m`                | string      | No       | Optional memo (max 500 characters).                             |

**Skill UID** is the sequence number of this `register` message.

#### Update

```json
{
  "p": "hcs-26",
  "op": "update",
  "uid": "42",
  "publisher": "0.0.78910",
  "metadata": {
    "name": "PDF Processing",
    "description": "Extract and clean PDF text",
    "author": "Example Labs",
    "license": "Apache-2.0",
    "tags": ["pdf", "text", "extraction", "ocr"]
  },
  "m": "update tags"
}
```

| Field      | Type   | Required | Description                                          |
| ---------- | ------ | -------- | ---------------------------------------------------- |
| `uid`      | string | Yes      | Sequence number of the original register message.    |
| `metadata` | object | No       | Updated metadata object.                             |
| `publisher`| string | No       | Updated publisher account ID or UAID.                |

#### Delete

```json
{
  "p": "hcs-26",
  "op": "delete",
  "uid": "42",
  "m": "remove skill"
}
```

| Field | Type   | Required | Description                                       |
| ----- | ------ | -------- | ------------------------------------------------- |
| `uid` | string | Yes      | Sequence number of the original register message. |

### Version Registry Operations

Version registry messages extend HCS-2 and use `p: "hcs-26"`.

#### Register (New Version)

```json
{
  "p": "hcs-26",
  "op": "register",
  "skill_uid": 42,
  "version": "1.0.0",
  "manifest_hcs1": "hcs://1/0.0.33333",
  "checksum": "sha256:...",
  "status": "active",
  "m": "initial release"
}
```

| Field           | Type   | Required | Description                                                     |
| --------------- | ------ | -------- | --------------------------------------------------------------- |
| `skill_uid`     | number | Yes      | Sequence number of the discovery registry register message.     |
| `version`       | string | Yes      | Semantic version string.                                        |
| `manifest_hcs1` | string | Yes      | HRL to the `SKILL.json` manifest on HCS-1.                       |
| `checksum`      | string | No       | Hash of the manifest file (recommended).                        |
| `status`        | string | No       | `active`, `deprecated`, or `yanked` (default `active`).          |

#### Update

```json
{
  "p": "hcs-26",
  "op": "update",
  "uid": "7",
  "status": "deprecated",
  "m": "superseded by 1.1.0"
}
```

| Field   | Type   | Required | Description                                       |
| ------- | ------ | -------- | ------------------------------------------------- |
| `uid`   | string | Yes      | Sequence number of the version register message.  |
| `status`| string | No       | Updated status.                                   |

#### Delete

```json
{
  "p": "hcs-26",
  "op": "delete",
  "uid": "7",
  "m": "remove version"
}
```

### Skill Manifest: `SKILL.json`

Each skill version MUST reference a `SKILL.json` manifest stored on HCS-1. The manifest defines the skill folder and file mappings.

**Required fields**

- `name` (string)
- `description` (string)
- `version` (string, semver)
- `license` (string, SPDX ID or `proprietary`)
- `author` (string or object)
- `files` (array of file entries)
- `files` MUST include an entry for `SKILL.md` at the root (`path` = `SKILL.md`).

**Optional fields**

- `tags` (string[])
- `homepage` (string, URL)
- `languages` (string[])
- `entrypoints` (array of runnable scripts)

**File entry schema**

- `path` (string, relative path)
- `hrl` (string, HCS-1 HRL)
- `sha256` (string, hex)
- `mime` (string)

**Example manifest**

```json
{
  "name": "PDF Processing",
  "description": "Extract and clean PDF text",
  "version": "1.0.0",
  "license": "Apache-2.0",
  "author": "Example Labs",
  "tags": ["pdf", "text"],
  "languages": ["python"],
  "entrypoints": [
    { "path": "scripts/extract.py", "language": "python", "args": ["--fast"] }
  ],
  "files": [
    {
      "path": "SKILL.md",
      "hrl": "hcs://1/0.0.44444",
      "sha256": "...",
      "mime": "text/markdown"
    },
    {
      "path": "scripts/extract.py",
      "hrl": "hcs://1/0.0.55555",
      "sha256": "...",
      "mime": "text/x-python"
    },
    {
      "path": "references/REFERENCE.md",
      "hrl": "hcs://1/0.0.66666",
      "sha256": "...",
      "mime": "text/markdown"
    }
  ]
}
```

### Skill Folder Conventions

Skills are logical folders represented by `files[].path` entries. Paths MUST be normalized and MUST NOT contain `..`.

Optional directories:

- `scripts/` for executable code
- `references/` for additional documentation and forms
- `assets/` for static resources

The file `SKILL.md` MUST be included at the root to describe the skill in Agent Skills format. Its frontmatter `name`, `description`, and `version` SHOULD match the manifest values.

### Metadata Schema (Discovery Registry)

The discovery registry metadata MUST follow this schema:

**Required**

- `name` (string)
- `description` (string)
- `author` (string or `{ name, contact, url }`)
- `license` (string, SPDX ID or `proprietary`)

**Optional**

- `tags` (string[])
- `homepage` (string URL)
- `icon_hcs1` (string HRL)
- `languages` (string[])
- `capabilities` (string[])

### Retrieval Rules

1. Discover skills from the discovery registry topic.
2. The skill identifier is the sequence number of the discovery `register` message.
3. Fetch the per-skill version registry using `version_registry`.
4. Select the highest semantic version with `status = active`.
5. Retrieve `SKILL.json` from `manifest_hcs1` and load any referenced files as needed.

### Reputation Architectures (Informative)

Implementations MAY adopt one of the following reputation architectures. Each uses an HCS-2 indexed topic with type enum `2`.

#### Option A: Signed Usage Receipts

- **Schema**: `skill_uid`, `version`, `rater_id`, `agent_id`, `score`, `receipt_hash`, `sig`
- **Reliability**: requires cryptographic signatures and replay protection; indexers SHOULD enforce minimum receipt counts.

#### Option B: Stake-Backed Attestations with Dispute

- **Schema**: `skill_uid`, `version`, `claim`, `score`, `stake_ref`
- **Reliability**: rater posts a bond; disputes resolved via HCS-8 poll; losing side is slashed.

#### Option C: Multi-Indexer Reputation Consensus

- **Schema**: `skill_uid`, `version`, `score`, `indexer_id`, `sig`, `snapshot_hash`
- **Reliability**: clients require >= N of M matching signed snapshots.

### Validation Rules

- `p` MUST equal `hcs-26`.
- `op` MUST be one of `register`, `update`, `delete`, `migrate`.
- `version_registry` and `manifest_hcs1` MUST be valid HCS topic IDs or HRLs as specified.
- `skill_uid` MUST be a valid sequence number from the discovery registry topic.
- `version` MUST be a valid semantic version.
- `SKILL.json` MUST be valid JSON and include all required fields.
- `SKILL.md` MUST be present in the manifest `files` list with `path` set to `SKILL.md`.

## Rationale

HCS-26 minimizes new infrastructure by combining HCS-2 registries with HCS-1 file storage and a manifest for folder structure. Using the discovery registry sequence number as the skill identifier eliminates external ID allocation while preserving determinism within a registry.

## Backwards Compatibility

HCS-26 is additive and does not modify existing HCS standards. Legacy clients may ignore HCS-26 topics without impact.

## Security Considerations

- Clients SHOULD verify manifest and file hashes before execution.
- Scripts SHOULD be executed in sandboxed environments.
- Discovery registry operators SHOULD monitor for malicious content and rely on `yanked` or `delete` updates.

## Privacy Considerations

Skill manifests and files are public once written to HCS. Publishers SHOULD avoid embedding sensitive data in skills or metadata.

## Test Vectors

See examples in the Discovery and Version Registry sections. Implementations SHOULD validate:

- Skill UID derivation from discovery register sequence number.
- Manifest hash verification and path normalization.
- Status transitions and version selection logic.

## Conformance

A conforming HCS-26 publisher MUST:

- Register skills in a discovery registry.
- Publish a per-skill version registry.
- Store `SKILL.json` and all referenced files on HCS-1.

A conforming HCS-26 client MUST:

- Resolve skill UIDs via discovery registry sequence numbers.
- Retrieve and validate the manifest.
- Respect `status` and version selection rules.

## References

- [HCS-1: File Management](./hcs-1.md)
- [HCS-2: Topic Registries](./hcs-2.md)
- [Agent Skills](https://agentskills.io)

## Governance Record (fill at publication)

- Poll topic: hcs://8/<topicId> (or Mirror Node link)
- Outcome: PASS | FAIL on YYYY-MM-DD (UTC)
- Reference: <txn id or final tally link>

## License

This document is licensed under Apache-2.0.
