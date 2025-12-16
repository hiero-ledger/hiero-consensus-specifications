---
description: A formal set of definitions used within the Hiero Consensus Standards (HCS) specifications. These are NOT standards and should be used for reference only.
---

import Link from '@docusaurus/Link';

# Definitions

<div className="definitions-container">

<p className="definitions-intro">A formal set of definitions used within the Hiero Consensus Standards (HCS) specifications.<br />These are <strong>NOT</strong> standards and should be used for reference only.</p>

<div className="definitions-list">

## Guarded Registries

Topic Ids which are managed by the Hashgraph Online Council for the convenience of the ecosystem. These topics should be private topics to ensure proper validation of messages. <Link to="/docs/standards/hcs-2">HCS</Link> will cover this idea in detail.

## Hashgraph Resource Locator

The protocol for finding data inscribed on the Hedera Consensus Service, following the Hashgraph Consensus Standards. An HRL locates a resource. The format for an HRL is as follows. A formal future HCS will cover all cases for HRLs.
- Format: `hcs://{protocol_standard}/{resource_id}`
- `protocol_number` should always be a valid HCS standard, eg `1`
- `resource_id` should always be a valid <Link to="#resource-conceptual-addresses">Resource</Link>
- Example: `hcs://1/0.0.123456`

## Hashsite

A fully rendered decentralized application, composed of valid HTML Markup, JavaScript, and CSS which is inscribed following HCS standards.

## Public Registries

Topic Ids which are managed by the community for the convenience of the ecosystem. These topics are public and not managed by a specific entity. Proceed with caution when consuming data inscribed on these topics.

## Resource (Conceptual Addresses)

A resource is any piece of data on Hedera that is referenced by an entity id on Hedera / Hashgraphs. Valid entities are Topic IDs.

## Topic Memos

Topic memos are structured strings attached to Hedera Consensus Service (HCS) topics that define the topic's purpose, configuration, and behavior. They serve as metadata that applications can use to understand how to interact with the topic.

<div className="memo-details">

**Standard Format:** `hcs-{standard_number}:{parameter1}:{parameter2}:...`
- `standard_number`: The HCS standard that defines the topic's behavior (e.g., 2, 6, 10, 13)
- Parameters vary by standard but follow consistent patterns

**Common Parameters:**
- **Indexed Flag** - Boolean value (0/1) indicating if all messages need to be processed (0) or only the latest message matters (1)
  - `hcs-2:0:86400` (indexed) | `hcs-6:non-indexed:86400` (non-indexed)
- **TTL (Time-to-Live)** - Duration in seconds that clients should cache topic data
  - `hcs-2:0:86400` (cache for 24 hours)
- **References** - Some standards include references to other topics
  - `hcs-13:0:86400:0.0.123456` (references schema topic 0.0.123456)

**Standard-Specific Examples:**
- **HCS-2 (Registry):** `hcs-2:0:86400` - indexed registry with 24-hour cache
- **HCS-6 (Dynamic Hashinals):** `hcs-6:non-indexed:86400` - non-indexed with 24-hour cache
- **HCS-10 (AI Agent Communication):** `hcs-10:inbound:0.0.123456` - inbound topic for agent 0.0.123456

**Purpose:** Topic memos enable:
- Self-documentation of topics
- Automatic discovery of topic functionality
- Consistent interpretation across different applications
- Configuration of caching and processing behavior

<div className="note">Applications should validate topic memos against the relevant HCS standard before processing topic messages to ensure correct interpretation and handling.</div>

</div>

</div>

</div>

<style jsx>{`
  /* Apply gradient to the main title */
  h1 {
    background: linear-gradient(90deg, #5599fe 0%, #b56cff 50%, #48df7b 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    font-weight: 700;
    font-size: 2.5rem;
  }

  .definitions-container {
    max-width: 900px;
    margin: 0 auto;
    margin-top: -2rem; /* Reduce gap after h1 */
    position: relative;
  }

  /* Add subtle background pattern */
  .definitions-container::before {
    content: '';
    position: absolute;
    top: -100px;
    left: -50%;
    right: -50%;
    height: 300px;
    background: radial-gradient(ellipse at center, rgba(85, 153, 254, 0.05) 0%, transparent 70%);
    z-index: -1;
    pointer-events: none;
  }

  .definitions-intro {
    text-align: left;
    color: var(--ifm-color-emphasis-700);
    margin-top: -3.5rem !important;
    margin-bottom: 0.75rem;
    font-size: 1.05rem;
    line-height: 1.7;
  }

  .definitions-list {
    /* No need for max-width here since parent container handles it */
  }

  .definitions-list h2 {
    font-size: 1.25rem;
    margin-top: 1.5rem;
    margin-bottom: 0.5rem;
    padding-bottom: 0.25rem;
    border-bottom: 1px solid var(--ifm-color-emphasis-200);
    position: relative;
    transition: all 0.3s ease;
  }

  /* Add gradient accent on hover */
  .definitions-list h2::after {
    content: '';
    position: absolute;
    bottom: -1px;
    left: 0;
    width: 0;
    height: 2px;
    background: linear-gradient(90deg, #5599fe 0%, #b56cff 50%, #48df7b 100%);
    transition: width 0.3s ease;
  }

  .definitions-list h2:hover::after {
    width: 100px;
  }

  .definitions-list h2:first-child {
    margin-top: 0;
  }

  .definitions-list ul {
    margin: 0.25rem 0 0.5rem 1.25rem;
    padding: 0;
  }

  .definitions-list li {
    margin: 0.125rem 0;
    line-height: 1.5;
  }

  .definitions-list code {
    background: var(--ifm-code-background);
    padding: 0.125rem 0.25rem;
    border-radius: 0.25rem;
    font-size: 0.85rem;
  }

  .memo-details {
    margin-top: 0.5rem;
    margin-left: 0;
    padding-left: 1rem;
    border-left: 2px solid var(--ifm-color-emphasis-200);
    position: relative;
  }

  /* Gradient border effect */
  .memo-details::before {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    bottom: 0;
    width: 2px;
    background: linear-gradient(180deg, #5599fe 0%, #b56cff 50%, #48df7b 100%);
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  .memo-details:hover::before {
    opacity: 1;
  }

  .memo-details > p {
    margin: 0.5rem 0;
    line-height: 1.6;
  }

  .memo-details strong {
    color: var(--ifm-color-primary);
    font-weight: 600;
  }

  /* Style code blocks with gradient shadow */
  .definitions-list code {
    background: var(--ifm-code-background);
    padding: 0.125rem 0.25rem;
    border-radius: 0.25rem;
    font-size: 0.85rem;
    transition: all 0.2s ease;
  }

  .definitions-list code:hover {
    box-shadow: 0 2px 8px rgba(85, 153, 254, 0.15);
    transform: translateY(-1px);
  }

  .memo-details ul {
    margin: 0.25rem 0 0.75rem 1.25rem;
  }

  .memo-details li {
    margin: 0.125rem 0;
  }

  .note {
    margin-top: 0.75rem;
    padding: 0.75rem;
    background: var(--ifm-color-info-contrast-background);
    border-left: 3px solid var(--ifm-color-info);
    border-radius: 0.25rem;
    font-size: 0.85rem;
    line-height: 1.5;
    position: relative;
    overflow: hidden;
  }

  /* Gradient accent effect */
  .note::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 1px;
    background: linear-gradient(90deg, #5599fe 0%, #b56cff 50%, #48df7b 100%);
    opacity: 0.5;
  }

  /* Add hover effect for list items */
  .definitions-list li {
    transition: color 0.2s ease;
  }

  .definitions-list li:hover {
    color: var(--ifm-color-primary);
  }


  /* Dark mode adjustments */
  [data-theme='dark'] .definitions-list h2 {
    border-bottom-color: var(--ifm-color-emphasis-300);
  }

  [data-theme='dark'] .memo-details {
    border-left-color: var(--ifm-color-emphasis-400);
  }

  [data-theme='dark'] .note {
    background: rgba(66, 153, 225, 0.1);
  }

`}</style>
