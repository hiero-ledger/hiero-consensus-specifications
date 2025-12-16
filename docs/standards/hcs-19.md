---
title: "HCS‑19 Standard: AI Agent Privacy Compliance"
description: "The HCS‑19 standard defines a comprehensive, ISO/IEC TS 27560‑aligned framework
for AI agents to document and manage privacy compliance on the Hedera Hashgraph through
HCS topics, enabling auditable consent management, data‑processing records, privacy‑rights
fulfilment and regulatory compliance for laws such as GDPR, CCPA and DDP."
sidebar_position: 19
---

# HCS‑19 Standard: AI Agent Privacy Compliance

### Status: Draft

### Version: 1.0 (aligns with ISO/IEC TS 27560:2023)

### Table of Contents

<!-- omit in toc -->

* [Authors](#authors)
* [Abstract](#abstract)
* [Motivation](#motivation)
* [Specification](#specification)

  * [Architecture Overview](#architecture-overview)
  * [Topic System](#topic-system)

    * [Topic Types and Formats](#topic-types-and-formats)
    * [Topic Memo Formats](#topic-memo-formats)
  * [Operation Reference](#operation-reference)

    * [Consent Management Operations](#consent-management-operations)
    * [Data Processing Operations](#data-processing-operations)
    * [Privacy Rights Operations](#privacy-rights-operations)
    * [Audit and Compliance Operations](#audit-and-compliance-operations)
  * [Privacy Compliance Schema](#privacy-compliance-schema)

    * [User Consent Record](#user-consent-record)
    * [Data Processing Activity](#data-processing-activity)
    * [Privacy Rights Request](#privacy-rights-request)
    * [Compliance Audit Entry](#compliance-audit-entry)
  * [Legal Framework Integration](#legal-framework-integration)

    * [GDPR Compliance Fields](#gdpr-compliance-fields)
    * [CCPA Compliance Fields](#ccpa-compliance-fields)
    * [DDP Compliance Fields](#ddp-compliance-fields)
  * [Integration with HCS‑11 Profiles](#integration-with-hcs-11-profiles)
  * [Large Data Handling](#large-data-handling)
  * [Retention and Archival](#retention-and-archival)
* [Implementation Workflow](#implementation-workflow)

  * [Step 1: Privacy Topic Setup](#step-1-privacy-topic-setup)
  * [Step 2: Agent Registration](#step-2-agent-registration)
  * [Step 3: User Consent Collection](#step-3-user-consent-collection)
  * [Step 4: Ongoing Compliance Monitoring](#step-4-ongoing-compliance-monitoring)
* [Security Considerations](#security-considerations)
* [Examples](#examples)

  * [Example 1: User Consent for Customer‑Service Agent](#example-1-user-consent-for-customer-service-agent)
  * [Example 2: Data‑Processing Activity Logging](#example-2-data-processing-activity-logging)
  * [Example 3: GDPR Data‑Subject Access Request](#example-3-gdpr-data-subject-access-request)
  * [Example 4: CCPA Do‑Not‑Sell Request](#example-4-ccpa-do-not-sell-request)
  * [Example 5: Compliance Audit Trail](#example-5-compliance-audit-trail)
* [Conclusion](#conclusion)

---

## Authors

* **[Roy Smith](https://www.linkedin.com/company/privacycheq)**
* **[Andrew Smith](https://www.linkedin.com/in/andrew-hyde-smith/)**

## Abstract

HCS‑19 defines a comprehensive framework for AI agents to document and manage
privacy compliance through Hedera Consensus Service (HCS) topics. The
ISO/IEC TS 27560:2023 aligned standard enables auditable consent management,
data‑processing records, privacy‑rights fulfilment and regulatory‑compliance
documentation for major privacy laws (GDPR, CCPA, DDP). By leveraging immutable,
timestamped records on HCS and a consent‑record structure compatible with
ISO 27560, AI agents can demonstrate compliance, issue verifiable consent
receipts and build user trust through transparent privacy practices.

## Motivation

As AI agents increasingly handle personal data, robust privacy‑compliance
documentation is critical. Current systems often lack transparency,
immutability and auditability demanded by modern regulations. HCS‑19 addresses
these challenges by:

**Regulatory Compliance** – Meets complex privacy‑law obligations with unified,
ISO‑standard consent records.

**User Trust & Transparency** – Immutable HCS records provide unprecedented
visibility into data practices.

**Auditable Consent Management** – Tamper‑proof recording and lifecycle
tracking of each consent instance.

**Automated Compliance Monitoring** – Real‑time logging and reporting of
compliance activities.

**Decentralised Privacy** – Removes single points of failure through Hedera’s
distributed ledger.

## Specification

### Architecture Overview

HCS‑19 leverages Hedera Consensus Service to create a complete privacy‑compliance
documentation system. Components:

```mermaid
graph TB
    subgraph "HCS-19 Privacy Compliance Documentation System"
        A[AI Agent] --> B[HCS-11 Profile Registration]
        B --> C{Privacy Compliance Topics}
        
        C --> D[Consent Management Topic<br/>Type: 0]
        C --> E[Data Processing Registry Topic<br/>Type: 1]
        C --> F[Privacy Rights Registry Topic<br/>Type: 2]
        C --> G[Compliance Audit Topic<br/>Type: 3]
        
        D --> H[User Consent Records<br/>ISO/IEC TS 27560 Aligned]
        E --> I[Processing Activity Logs]
        F --> J[Rights Request Handling]
        G --> K[Audit Trail Generation]
        
        H --> L[Legal Framework Mapping]
        I --> L
        J --> L
        K --> L
        
        L --> M[GDPR Compliance]
        L --> N[CCPA Compliance]
        L --> O[DDP Compliance]
        
        P[Retention Management] --> H
        P --> I
        P --> J
        P --> K
        
        Q[Large Data Handling<br/>via HCS-1] --> H
        Q --> I
        Q --> J
        Q --> K
        
        R[User] --> A
        S[Regulators/Auditors] --> K
        T[Data Protection Officer] --> G
    end
    
    style A fill:#e1f5fe
    style H fill:#f3e5f5
    style I fill:#e8f5e8
    style J fill:#fff3e0
    style K fill:#fce4ec
    style L fill:#f1f8e9
    style P fill:#f5f5f5
    style Q fill:#e3f2fd
```

1. **Privacy‑Compliance Topics** – Dedicated HCS topics for consent, processing,
   rights and audit logs.
2. **Agent Registration** – HCS‑11 profiles identify compliant AI agents.
3. **Legal Framework Mapping** – Built‑in support for multiple privacy laws.
4. **Audit‑Trail Generation** – Comprehensive ledger of all privacy activities.
5. **Retention Management** – Automated lifecycle rules for retention/deletion.
6. **Standardised Consent Records** – JSON structure fully conformant with
   ISO/IEC TS 27560:2023 for interoperability and verifiability.

### Topic System

#### Topic Types and Formats

| Topic Type                   | Description                                 | Key Configuration             | Enum |
| ---------------------------- | ------------------------------------------- | ----------------------------- | ---- |
| **Consent Management**       | Records user consents and withdrawals       | Submit key (agent)            | 0    |
| **Data‑Processing Registry** | Logs data‑processing events                 | Submit key (agent)            | 1    |
| **Privacy‑Rights Registry**  | Handles rights requests / responses         | Submit key (agent)            | 2    |
| **Compliance Audit**         | Stores compliance monitoring and audit logs | Submit key (agent + auditors) | 3    |

#### Topic Memo Formats

**Consent Management Memo**

```
hcs-19:0:\{ttl\}:\{topic_type\}:\{agent_account_id\}:\{jurisdiction\}
```

Other memo formats follow the same pattern, replacing \{topic_type\} with the appropriate enum as required.


### Operation Reference

#### Consent Management Operations

| Operation           | Description                                     |
| ------------------- | ----------------------------------------------- |
| `consent_granted`   | Record user consent for data processing         |
| `consent_withdrawn` | Record withdrawal of previously‑granted consent |
| `consent_updated`   | Record modified consent preferences             |
| `consent_verified`  | Verify current consent status                   |

**Consent Granted Example**

```json
{
  "p": "hcs-19",
  "op": "consent_granted",
  "operator_id": "0.0.12345@0.0.4456",
  "user_id": "user_12345",
  "consent_id": "consent_67890",
  "purposes": ["service_delivery", "personalization"],
  "legal_basis": "consent",
  "jurisdiction": "EU",
  "consent_method": "explicit_opt_in",
  "data_types": ["personal_identifiers", "communication_data"],
  "retention_period": "2_years",
  "notice_reference": "hcs://1/0.0.999888#",
  "timestamp": "2025-01-15T10:30:00Z",
  "m": "User consent granted for customer service interactions"
}
```

#### Data Processing Operations

| Operation              | Description                             |
| ---------------------- | --------------------------------------- |
| `processing_started`   | Log start of a processing activity      |
| `processing_completed` | Log completion of a processing activity |
| `data_shared`          | Record data sharing with third parties  |
| `data_deleted`         | Record deletion of personal data        |

#### Privacy Rights Operations

| Operation                 | Description                              |
| ------------------------- | ---------------------------------------- |
| `rights_request`          | Log incoming privacy‑rights request      |
| `rights_fulfilled`        | Record fulfilment of a rights request    |
| `access_provided`         | Log provision of data access to the user |
| `rectification_completed` | Record completion of data rectification  |
| `erasure_completed`       | Record completion of data erasure        |

#### Audit and Compliance Operations

| Operation            | Description                       |
| -------------------- | --------------------------------- |
| `compliance_check`   | Record automated compliance check |
| `violation_detected` | Log detected compliance violation |
| `audit_initiated`    | Record start of an audit          |
| `audit_completed`    | Record completion of an audit     |

### Privacy Compliance Schema

#### User Consent Record

```jsonc
{
  "consent_id": "string",           // Unique consent identifier
  "user_id": "string",              // Pseudonymised user identifier
  "agent_id": "string",             // AI agent (PII controller) account ID
  "jurisdiction": "string",         // ISO‑3166‑1‑alpha‑2 code
  "legal_basis": "string",          // Lawful basis for processing
  "purposes": ["string"],           // Processing purposes
  "data_types": ["string"],         // Categories of data processed
  "consent_method": "string",       // How consent was obtained
  "consent_timestamp": "string",    // ISO 8601 timestamp
  "expiry_date": "string",          // Consent expiry date (if any)
  "retention_period": "string",     // Data‑retention period
  "withdrawal_method": "string",    // How user can withdraw consent
  "status": "string",               // "active" | "withdrawn"
  "notice_reference": "string",     // URI / HCS-1 ref of privacy notice version
  "granular_permissions": {
    "analytics": "boolean",
    "marketing": "boolean",
    "personalization": "boolean"
  }
}
```

#### Data Processing Activity

```jsonc
{
  "processing_id": "string",
  "user_id": "string",
  "agent_id": "string",
  "purpose": "string",
  "legal_basis": "string",
  "data_types": ["string"],
  "processing_method": "string",
  "duration": "string",
  "third_parties": ["string"],
  "security_measures": ["string"],
  "start_timestamp": "string",
  "end_timestamp": "string",
  "compliance_status": "string"
}
```

#### Privacy Rights Request

```jsonc
{
  "request_id": "string",
  "user_id": "string",
  "agent_id": "string",
  "request_type": "string",
  "jurisdiction": "string",
  "legal_basis": "string",
  "request_timestamp": "string",
  "verification_method": "string",
  "fulfillment_method": "string",
  "expected_completion": "string",
  "actual_completion": "string",
  "response_method": "string",
  "compliance_notes": "string"
}
```

#### Compliance Audit Entry

```jsonc
{
  "audit_id": "string",
  "agent_id": "string",
  "audit_type": "string",
  "auditor_id": "string",
  "audit_scope": ["string"],
  "audit_period": { "start_date": "string", "end_date": "string" },
  "findings": ["string"],
  "compliance_score": "number",
  "violations": ["string"],
  "recommendations": ["string"],
  "follow_up_required": "boolean",
  "follow_up_date": "string",
  "audit_timestamp": "string"
}
```

### Legal Framework Integration

#### GDPR Compliance Fields

```jsonc
{
  "gdpr_lawful_basis": "string",
  "special_category_basis": "string",
  "data_controller": "string",
  "data_processor": "string",
  "dpo_contact": "string",
  "transfer_mechanism": "string",
  "retention_justification": "string",
  "automated_decision_making": "boolean",
  "profiling_activities": ["string"]
}
```

#### CCPA Compliance Fields

```jsonc
{
  "business_purpose": "string",
  "commercial_purpose": "string",
  "sale_opt_out": "boolean",
  "categories_sold": ["string"],
  "categories_disclosed": ["string"],
  "third_party_recipients": ["string"],
  "retention_justification": "string",
  "consumer_rights_provided": ["string"]
}
```

#### DDP Compliance Fields

```jsonc
{
  "collection_method": "string",
  "notification_provided": "boolean",
  "purpose_limitation": "boolean",
  "data_minimization": "boolean",
  "accuracy_measures": ["string"],
  "storage_limitation": "string",
  "security_measures": ["string"],
  "accountability_measures": ["string"]
}
```

### Integration with HCS‑11 Profiles

AI agents **MUST** maintain a valid HCS‑11 profile including privacy‑compliance
metadata:

```jsonc
{
  "version": "1.0",
  "type": 1,
  "display_name": "Privacy‑Compliant Customer Service Agent",
  "privacy_compliance": {
    "standards": ["gdpr", "ccpa", "ddp"],
    "jurisdictions": ["EU", "US-CA", "US"],
    "consent_topic_id": "0.0.789101",
    "processing_topic_id": "0.0.789102",
    "rights_topic_id": "0.0.789103",
    "audit_topic_id": "0.0.789104",
    "dpo_contact": "dpo@example.com",
    "privacy_policy_url": "https://example.com/privacy",
    "retention_policy": "2_years_default"
  },
  "inboundTopicId": "0.0.123456",
  "outboundTopicId": "0.0.123457"
}
```

### Large Data Handling

Where consent‑related documents (e.g., full privacy notice) exceed HCS message
size, reference the content stored under **HCS‑1**:

```jsonc
{
  "p": "hcs-19",
  "op": "consent_granted",
  "operator_id": "0.0.123456",
  "user_id": "user_12345",
  "consent_data": "hcs://1/0.0.999888",
  "m": "Large notice stored via HCS‑1"
}
```

Referencing the exact notice version satisfies ISO 27560’s requirement to retain
the presented notice alongside the consent record.

### Retention and Archival

HCS‑19 supports automated retention management:

```jsonc
{
  "p": "hcs-19",
  "op": "retention_check",
  "operator_id": "0.0.123456",
  "records_reviewed": 1500,
  "records_archived": 150,
  "records_deleted": 25,
  "retention_policies_applied": ["2_year_default", "1_year_marketing"],
  "compliance_status": "compliant",
  "next_review_date": "2025-04-15T10:00:00Z",
  "m": "Quarterly retention review completed"
}
```

## Implementation Workflow

### Step 1: Privacy Topic Setup

1. Create four HCS topics (consent, processing, rights, audit).
2. Apply memo formats and set submit keys.
3. Register topic IDs in the agent’s HCS‑11 profile.

### Step 2: Agent Registration

1. Update HCS‑11 profile with privacy‑compliance block.
2. List supported jurisdictions and standards.
3. Provide controller and DPO contact details.

### Step 3: User Consent Collection

1. Present a clear privacy notice (ISO 29184 guidance) and collect explicit
   consent via UI controls.
2. Log consent events (`consent_granted`, etc.) on the Consent topic.
3. Verify active consent prior to any data processing.
4. **Issue a Consent Receipt** – after consent is recorded, provide the user a
   machine‑ and human‑readable receipt summarising consent ID, controller,
   purposes, notice reference and timestamp. The receipt includes the HCS
   transaction ID so users (or auditors) can independently verify authenticity.

### Step 4: Ongoing Compliance Monitoring

1. Record processing events in real time.
2. Respond to privacy‑rights requests within statutory deadlines.
3. Conduct and log periodic compliance audits.
4. Execute automated retention/deletion routines.

## Security Considerations

1. **Data Minimisation** – only include data necessary for compliance.
2. **Pseudonymisation** – use hashed or tokenised user IDs.
3. **Access Control & Authenticity** – restrict topic submission to authorised
   keys and digitally sign messages to guarantee integrity and origin.
4. **Encryption** – encrypt sensitive payloads before submission.
5. **Retention Limits** – automatically purge expired records.
6. **Audit Logging** – maintain a complete, immutable log of compliance
   activities.

## Examples

### Example 1: User Consent for Customer‑Service Agent

```jsonc
{
  "p": "hcs-19",
  "op": "consent_granted",
  "operator_id": "0.0.123456",
  "user_id": "usr_hash_abc123",
  "consent_id": "consent_2025_001",
  "purposes": ["customer_support", "service_improvement"],
  "legal_basis": "consent",
  "jurisdiction": "EU",
  "consent_method": "explicit_checkbox",
  "data_types": ["contact_information", "conversation_history"],
  "retention_period": "2_years",
  "gdpr_lawful_basis": "article_6_1_a",
  "withdrawal_method": "email_or_chat",
  "notice_reference": "hcs://1/0.0.999888#v1.3",
  "status": "active",
  "timestamp": "2025-01-15T10:30:00Z",
  "m": "Customer granted consent for AI customer service"
}
```

```mermaid
sequenceDiagram
    participant U as User
    participant A as AI Agent<br/>(0.0.123456)
    participant CT as Consent Topic<br/>(0.0.789101)
    participant HCS as Hedera Consensus Service
    participant DPO as Data Protection Officer
    
    Note over U,DPO: Example 1: User Consent for Customer Service Agent
    
    U->>A: Initiates customer service interaction
    A->>U: Presents privacy notice (HCS-1 ref: 0.0.999888#v1.3)
    A->>U: Displays consent form with explicit checkboxes
    
    Note over U,A: Purposes: customer_support, service_improvement<br/>Data Types: contact_information, conversation_history
    
    U->>A: Grants consent via explicit checkbox
    A->>CT: Submit consent_granted operation
    
    Note over CT: {<br/>"op": "consent_granted",<br/>"user_id": "usr_hash_abc123",<br/>"consent_id": "consent_2025_001",<br/>"legal_basis": "consent",<br/>"jurisdiction": "EU",<br/>"gdpr_lawful_basis": "article_6_1_a"<br/>}
    
    CT->>HCS: Record immutable consent transaction
    HCS-->>CT: Transaction ID & timestamp
    CT-->>A: Consent recorded successfully
    A->>U: Issue consent receipt with HCS transaction ID
    
    Note over A,DPO: Retention: 2 years<br/>Withdrawal: email_or_chat<br/>Status: active
    
    A->>A: Verify active consent before processing
    A->>U: Proceed with customer service interaction
```

### Example 2: Data‑Processing Activity Logging

```jsonc
{
  "p": "hcs-19",
  "op": "processing_started",
  "operator_id": "0.0.123456",
  "user_id": "usr_hash_abc123",
  "processing_id": "proc_2025_001",
  "purpose": "sentiment_analysis",
  "legal_basis": "consent",
  "data_types": ["conversation_text", "emotional_indicators"],
  "processing_method": "automated_nlp_analysis",
  "expected_duration": "immediate",
  "security_measures": ["encryption", "access_logging"],
  "timestamp": "2025-01-15T10:35:00Z",
  "m": "Started sentiment analysis for customer feedback"
}
```


```mermaid
sequenceDiagram
    participant U as User
    participant A as AI Agent<br/>(0.0.123456)
    participant PT as Processing Topic<br/>(0.0.789102)
    participant NLP as NLP Analysis Engine
    participant HCS as Hedera Consensus Service
    
    Note over U,HCS: Example 2: Data Processing Activity Logging
    
    U->>A: Provides customer feedback
    A->>A: Verify active consent exists
    
    A->>PT: Log processing_started operation
    Note over PT: {<br/>"op": "processing_started",<br/>"user_id": "usr_hash_abc123",<br/>"processing_id": "proc_2025_001",<br/>"purpose": "sentiment_analysis",<br/>"legal_basis": "consent",<br/>"data_types": ["conversation_text", "emotional_indicators"]<br/>}
    
    PT->>HCS: Record processing start transaction
    HCS-->>PT: Transaction ID & timestamp (2025-01-15T10:35:00Z)
    
    A->>NLP: Initiate automated NLP sentiment analysis
    Note over NLP: Processing Method: automated_nlp_analysis<br/>Security Measures: encryption, access_logging<br/>Expected Duration: immediate
    
    NLP->>NLP: Analyze conversation text for emotional indicators
    NLP-->>A: Return sentiment analysis results
    
    A->>PT: Log processing_completed operation
    Note over PT: {<br/>"op": "processing_completed",<br/>"processing_id": "proc_2025_001",<br/>"end_timestamp": "2025-01-15T10:35:15Z",<br/>"compliance_status": "compliant"<br/>}
    
    PT->>HCS: Record processing completion
    HCS-->>PT: Completion transaction logged
    
    A->>U: Provide enhanced customer service based on sentiment
```


### Example 3: GDPR Data‑Subject Access Request

```jsonc
{
  "p": "hcs-19",
  "op": "rights_request",
  "operator_id": "0.0.123456",
  "user_id": "usr_hash_abc123",
  "request_id": "sar_2025_001",
  "request_type": "data_access",
  "jurisdiction": "EU",
  "legal_basis": "gdpr_article_15",
  "request_method": "authenticated_email",
  "verification_status": "identity_verified",
  "expected_completion": "2025-02-14T10:00:00Z",
  "fulfillment_method": "secure_download_link",
  "timestamp": "2025-01-15T11:00:00Z",
  "m": "GDPR Article 15 subject access request received"
}
```


```mermaid
sequenceDiagram
    participant U as Data Subject
    participant A as AI Agent<br/>(0.0.123456)
    participant RT as Rights Topic<br/>(0.0.789103)
    participant DB as Data Storage
    participant HCS as Hedera Consensus Service
    participant DPO as Data Protection Officer
    
    Note over U,DPO: Example 3: GDPR Data Subject Access Request (Article 15)
    
    U->>A: Submits data access request via authenticated email
    A->>A: Verify user identity
    
    A->>RT: Log rights_request operation
    Note over RT: {<br/>"op": "rights_request",<br/>"user_id": "usr_hash_abc123",<br/>"request_id": "sar_2025_001",<br/>"request_type": "data_access",<br/>"jurisdiction": "EU",<br/>"legal_basis": "gdpr_article_15"<br/>}
    
    RT->>HCS: Record request transaction
    HCS-->>RT: Transaction ID & timestamp (2025-01-15T11:00:00Z)
    
    Note over A,DPO: Expected completion: 2025-02-14T10:00:00Z<br/>(30 days as per GDPR)
    
    A->>DB: Query all personal data for user
    DB-->>A: Return user's personal data
    
    A->>A: Compile data access report
    A->>A: Generate secure download link
    
    A->>RT: Log access_provided operation
    Note over RT: {<br/>"op": "access_provided",<br/>"request_id": "sar_2025_001",<br/>"fulfillment_method": "secure_download_link",<br/>"actual_completion": "2025-01-16T14:30:00Z"<br/>}
    
    RT->>HCS: Record fulfillment transaction
    HCS-->>RT: Fulfillment logged
    
    A->>U: Send secure download link via authenticated email
    U->>A: Download personal data report
    
    Note over U,DPO: Request fulfilled within GDPR timeframe<br/>Verification: identity_verified<br/>Method: authenticated_email
```

### Example 4: CCPA Do‑Not‑Sell Request

```jsonc
{
  "p": "hcs-19",
  "op": "rights_request",
  "operator_id": "0.0.123456",
  "user_id": "usr_hash_def456",
  "request_id": "dns_2025_001",
  "request_type": "do_not_sell",
  "jurisdiction": "US-CA",
  "legal_basis": "ccpa_section_1798_120",
  "request_method": "web_form",
  "verification_status": "verified",
  "ccpa_business_purpose": "customer_service_only",
  "sale_opt_out": true,
  "timestamp": "2025-01-15T14:20:00Z",
  "m": "CCPA Do Not Sell request processed"
}
```


```mermaid
sequenceDiagram
    participant C as California Consumer
    participant A as AI Agent<br/>(0.0.123456)
    participant RT as Rights Topic<br/>(0.0.789103)
    participant TS as Third-Party Services
    participant HCS as Hedera Consensus Service
    
    Note over C,HCS: Example 4: CCPA Do-Not-Sell Request (Section 1798.120)
    
    C->>A: Submits "Do Not Sell" request via web form
    A->>A: Verify consumer identity
    
    A->>RT: Log rights_request operation
    Note over RT: {<br/>"op": "rights_request",<br/>"user_id": "usr_hash_def456",<br/>"request_id": "dns_2025_001",<br/>"request_type": "do_not_sell",<br/>"jurisdiction": "US-CA",<br/>"legal_basis": "ccpa_section_1798_120"<br/>}
    
    RT->>HCS: Record request transaction
    HCS-->>RT: Transaction ID & timestamp (2025-01-15T14:20:00Z)
    
    Note over A: Verification Status: verified<br/>Request Method: web_form<br/>Business Purpose: customer_service_only
    
    A->>A: Update user preference: sale_opt_out = true
    A->>TS: Notify third parties to stop data sales
    
    TS-->>A: Confirm opt-out processed
    
    A->>RT: Log rights_fulfilled operation
    Note over RT: {<br/>"op": "rights_fulfilled",<br/>"request_id": "dns_2025_001",<br/>"ccpa_business_purpose": "customer_service_only",<br/>"sale_opt_out": true,<br/>"fulfillment_method": "immediate_processing"<br/>}
    
    RT->>HCS: Record fulfillment transaction
    HCS-->>RT: Fulfillment logged
    
    A->>C: Send confirmation of Do Not Sell processing
    
    Note over C,HCS: Consumer data sale stopped<br/>Third parties notified<br/>Preference permanently recorded
```

### Example 5: Compliance Audit Trail

```jsonc
{
  "p": "hcs-19",
  "op": "audit_completed",
  "operator_id": "0.0.123456",
  "audit_id": "audit_2025_q1",
  "audit_type": "internal_quarterly",
  "auditor_id": "auditor_internal_001",
  "audit_scope": ["consent_management", "data_processing", "rights_fulfillment"],
  "audit_period": {
    "start_date": "2025-01-01T00:00:00Z",
    "end_date": "2025-03-31T23:59:59Z"
  },
  "compliance_score": 98,
  "findings": ["consent_withdrawal_avg_response_2_hours", "100_percent_rights_fulfillment"],
  "violations": [],
  "recommendations": ["implement_real_time_consent_sync"],
  "follow_up_required": false,
  "timestamp": "2025-04-05T16:00:00Z",
  "m": "Q1 2025 privacy compliance audit completed"
}
```


```mermaid
flowchart TD
    subgraph "Q1 2025 Compliance Audit Process"
        A[Internal Auditor<br/>auditor_internal_001] --> B[Initiate Quarterly Audit<br/>audit_2025_q1]
        
        B --> C{Audit Scope}
        C --> D[Consent Management<br/>Review]
        C --> E[Data Processing<br/>Analysis]
        C --> F[Rights Fulfillment<br/>Assessment]
        
        D --> G[Review Consent Topic<br/>0.0.789101]
        E --> H[Review Processing Topic<br/>0.0.789102]
        F --> I[Review Rights Topic<br/>0.0.789103]
        
        G --> J[Analyze Period:<br/>2025-01-01 to 2025-03-31]
        H --> J
        I --> J
        
        J --> K{Audit Findings}
        K --> L[Consent Withdrawal<br/>Avg Response: 2 hours]
        K --> M[Rights Fulfillment<br/>100% completion rate]
        K --> N[No Violations<br/>Found]
        
        L --> O[Calculate Compliance Score<br/>98/100]
        M --> O
        N --> O
        
        O --> P[Generate Recommendations<br/>Real-time consent sync]
        
        P --> Q[Log audit_completed<br/>to Audit Topic 0.0.789104]
        
        Q --> R[Record to HCS<br/>2025-04-05T16:00:00Z]
        
        R --> S[Audit Trail Created<br/>No Follow-up Required]
    end
    
    style A fill:#e3f2fd
    style O fill:#e8f5e8
    style Q fill:#fce4ec
    style S fill:#f3e5f5
    style N fill:#e8f5e8
```

## Conclusion

HCS‑19 delivers an immutable, transparent and automated privacy‑compliance
framework for AI agents on Hedera. Alignment with **ISO/IEC TS 27560:2023**
ensures each consent record and user receipt follows a globally standardised
structure, improving interoperability, verifiability and regulatory trust. The
standard empowers proactive compliance, user control and auditable proof while
keeping implementation lightweight and decentralised—transforming privacy
governance from a burden into a competitive advantage.
