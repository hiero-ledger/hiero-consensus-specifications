# HashLinks - Actions, Blocks, and Assembly Standard

## Status: Draft

## Authors
- Kantorcodes [https://twitter.com/kantorcodes](https://twitter.com/kantorcodes)

## Abstract

HashLinks provides a framework for building interactive experiences on Hedera without the overhead of smart contracts. The standard combines WebAssembly modules for logic execution, WordPress Gutenberg blocks for UI components, and an assembly layer for composition.

Normative language in this document uses the key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY as described in RFC 2119.

## Motivation

Hedera's ecosystem needs better composability solutions. While smart contracts offer powerful capabilities, they're often expensive and complex for many use cases. HashLinks addresses this gap by providing:

1. Deterministic logic execution through WebAssembly without smart contract fees
2. Reusable UI components based on proven standards (Gutenberg)
3. On-chain verification without relying on centralized infrastructure
4. Seamless integration with existing Hedera wallets and tools

The standard consists of three interconnected layers:
- **[Actions](./actions.md)** - WASM modules for deterministic execution
- **[Blocks](./blocks.md)** - UI components for user interaction
- **[Assembly](./assembly.md)** - Composition layer for combining actions and blocks

For implementation details, see the **[Standards SDK HCS-12 Guide](/docs/libraries/standards-sdk/hcs-12/)** which documents the TypeScript/JavaScript implementation.

## Architecture Overview

The HashLinks architecture consists of three core components:

1. **Actions**: WebAssembly modules that execute business logic
2. **Blocks**: UI components built on the Gutenberg block standard
3. **Assembly**: Composition layer that binds actions to blocks

### Basic Architecture

The relationship between components:

```mermaid
graph TD
    A[Actions Layer - WASM Modules] -->|binds to| B[Blocks Layer - UI Components]
    B -->|composed via| C[Assembly Layer]
    C -->|creates| D[HashLink Experience]
    E[HCS-1 Storage] -->|stores files| A
    F[HCS-3 Resources] -->|loads resources in| B
    G[HCS-2 Topics] -->|foundation for| A
    G -->|foundation for| B
    G -->|foundation for| C
```

### Detailed Architecture

Complete system architecture including all dependencies:

```mermaid
flowchart TD
    subgraph "HashLinks Ecosystem"
        subgraph "HashLinks Architecture"
            ActionLayer["Actions Layer"]
            BlockLayer["Blocks Layer"]
            AssemblyLayer["Assembly Layer"]

            ActionLayer -->|provides logic for| BlockLayer
            BlockLayer -->|composed via| AssemblyLayer
            AssemblyLayer -->|creates| HashLinkExp["HashLink Experience"]
        end

        subgraph "Underlying Protocols"
            HCS1["HCS-1<br>File Storage"]
            HCS2["HCS-2<br>Topic Registries"]
            HCS3["HCS-3<br>Resource Loading"]

            HCS1 -->|stores| WASMFiles["WASM Modules"]
            HCS1 -->|stores| Templates["HTML Templates"]
            HCS1 -->|stores| Resources["CSS/JS Resources"]

            HCS2 -->|structures| ActionReg["Action Registry"]
            HCS2 -->|structures| BlockReg["Block Registry"]
            HCS2 -->|structures| AssemblyReg["Assembly Registry"]

            HCS3 -->|loads| Resources
        end

        WASMFiles -->|hosted in| ActionLayer
        Templates -->|referenced by| BlockLayer
        Resources -->|used by| BlockLayer

        ActionReg -->|organizes| ActionLayer
        BlockReg -->|organizes| BlockLayer
        AssemblyReg -->|organizes| AssemblyLayer
    end

    User(["End User"]) -->|interacts with| HashLinkExp
    Developer(["Developer"]) -->|creates| ActionLayer
    Developer -->|designs| BlockLayer
    Developer -->|composes| AssemblyLayer

    style ActionLayer fill:#f9d5e5,stroke:#333,stroke-width:2px
    style BlockLayer fill:#d5e5f9,stroke:#333,stroke-width:2px
    style AssemblyLayer fill:#d5f9e5,stroke:#333,stroke-width:2px
    style HashLinkExp fill:#f5f5f5,stroke:#333,stroke-width:2px
    style HCS1 fill:#ffe6cc,stroke:#333,stroke-width:1px
    style HCS2 fill:#e6ccff,stroke:#333,stroke-width:1px
    style HCS3 fill:#ccffe6,stroke:#333,stroke-width:1px
```

## Topic System

HashLinks leverages HCS-2 for registry management, following the principle of no admin keys to ensure decentralization.

### Topic Types

Registry structure and storage locations:

| Topic Type | Purpose | Configuration |
|------------|---------|---------------|
| **Action Registry** | Stores WASM module references | Non-indexed (latest version) |
| **Assembly Registry** | Complete HashLink configurations | Indexed (maintains history) |
| **HashLinks Registry** | Global directory of HashLinks | Non-indexed |
| **Block Storage** | Block definitions and templates | HCS-1 protocol |
| **Resource References** | Asset loading mechanism | HCS-3 protocol |

Registries SHOULD support HIP-991 fee collection for sustainability.

### Topic Memo Format

Following HCS-2 specification:

```
hcs-12:{indexed}:{ttl}:{type}:[additional parameters]
```

Where:
- `indexed` indicates whether all messages need to be read (0) or only the latest message (1)
- `ttl` specifies a time-to-live in seconds for caching
- `type` defines the component type using an enum value
- Additional parameters vary by topic type

**Type Enum Values**
| Type Enum | Component Type | Description |
|-----------|----------------|-------------|
| `0` | Action | WASM module registry (non-indexed, similar to HCS-6) |
| `1` | Assembly | Composition registry (indexed) |
| `2` | HashLinks | Global directory registry (non-indexed) |

**Action Registry Topic Memo**
```
hcs-12:1:60:0
```

**Assembly Registry Topic Memo**
```
hcs-12:0:60:1
```

**HashLinks Registry Topic Memo**
```
hcs-12:1:60:2
```

## WebAssembly Interface Stability

HashLinks implementations MUST store both the WASM binary and its JavaScript wrapper on-chain. This ensures consistent execution across different environments:

1. **JavaScript Wrapper Requirement** - The wasm-bindgen generated JavaScript must be inscribed alongside the WASM
2. **Version Tracking** - The `interface_version` field tracks the wasm-bindgen version used
3. **Stable Execution** - Both files together ensure deterministic execution regardless of client implementation

This approach addresses the inherent instability of WebAssembly imports and the complexity of wasm-bindgen's memory management.

## Operation Reference

This section defines all operations available in the HCS-12 protocol. Operations MUST be submitted as JSON messages to the appropriate registry topics.

### Action Registry Operations

| Operation | Description | Required Fields |
|-----------|-------------|-----------------|
| `register` | Register a new WASM action module | `p`, `op`, `t_id`, `hash`, `wasm_hash` |

#### Register Action

Registers a new WASM module with its metadata and verification hashes. Messages MUST include the required fields noted above and MAY include optional fields where specified:

```json
{
  "p": "hcs-12",
  "op": "register",
  "t_id": "0.0.123456",
  "hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "wasm_hash": "a1b2c3d4e5f6789012345678901234567890123456789012345678901234567890",
  "js_t_id": "0.0.123457",
  "js_hash": "5f6e7d8c9b0a1234567890123456789012345678901234567890123456789012",
  "interface_version": "0.2.92",
  "info_t_id": "0.0.123458",
  "validation_rules": {
    "amount": {
      "type": "number",
      "min": 0,
      "max": 1000000
    }
  },
  "m": "Token transfer action v1.0.0"
}
```

### Assembly Registry Operations

| Operation | Description | Required Fields |
|-----------|-------------|-----------------|
| `register` | Create a new assembly | `p`, `op`, `name`, `version` |
| `add-action` | Add an action to the assembly | `p`, `op`, `t_id`, `alias` |
| `add-block` | Add a block to the assembly | `p`, `op`, `block_t_id` |
| `update` | Update assembly metadata | `p`, `op` |

#### Register Assembly

Creates a new assembly that can combine actions and blocks:

```json
{
  "p": "hcs-12",
  "op": "register",
  "name": "defi-dashboard",
  "version": "1.0.0",
  "description": "Complete DeFi dashboard with swapping and staking",
  "tags": ["defi", "trading", "staking"],
  "author": "0.0.123456"
}
```

#### Add Action

Adds a WASM action to the assembly:

```json
{
  "p": "hcs-12",
  "op": "add-action",
  "t_id": "0.0.123456",
  "alias": "swap",
  "config": {
    "slippage": 0.5,
    "deadline": 300
  }
}
```

#### Add Block

Adds a UI block to the assembly:

```json
{
  "p": "hcs-12",
  "op": "add-block",
  "block_t_id": "0.0.234568",
  "actions": {
    "swap": "0.0.123456",
    "approve": "0.0.123457"
  },
  "attributes": {
    "title": "Quick Swap",
    "defaultToken": "HBAR"
  }
}
```

#### Update Assembly

Updates assembly metadata:

```json
{
  "p": "hcs-12",
  "op": "update",
  "description": "Updated DeFi dashboard with new features",
  "tags": ["defi", "trading", "staking", "lending"]
}
```

### HashLinks Registry Operations

| Operation | Description | Required Fields |
|-----------|-------------|-----------------|
| `register` | Register a HashLink in the global directory | `p`, `op`, `t_id`, `name` |

#### Register HashLink

Registers an assembly in the global HashLinks directory:

```json
{
  "p": "hcs-12",
  "op": "register",
  "t_id": "0.0.567890",
  "name": "Ultimate DeFi Dashboard",
  "description": "All-in-one DeFi management interface",
  "tags": ["defi", "featured"],
  "category": "finance",
  "author": "0.0.123456",
  "website": "https://example.com"
}
```

## Security Considerations

Key security requirements:

1. Hash Verification: Clients MUST verify WASM, JavaScript wrapper, and metadata hashes before execution.
2. Template Security: Templates MUST NOT include inline scripts; all resources MUST be loaded via HCS-3.
3. State Isolation: Blocks MUST be isolated by default, with explicit permission required for communication.
4. Action Validation: All actions MUST be registered with parameter validation and HTML sanitization.
5. Interface Verification: The JavaScript wrapper MUST match the WASM module interface version.

## Implementation Guide

### Step 1: Create Your WASM Action

```mermaid
sequenceDiagram
    participant Developer as Developer
    participant Hedera as Hedera
    participant Registry as Action Registry

    Developer->>Hedera: Create Action Registry Topic
    Note over Hedera: memo: hcs-12:1:60:0

    Developer->>Registry: Submit registration
    Note over Registry: {<br/>  "p": "hcs-12",<br/>  "op": "register",<br/>  "t_id": "0.0.123456",<br/>  "js_t_id": "0.0.123457",<br/>  "hash": "e3b0c44...",<br/>  "wasm_hash": "a1b2c3d4...",<br/>  "js_hash": "5f6e7d8c...",<br/>  "interface_version": "0.2.92"<br/>}
    Note over Registry: t_id points to WASM<br/>js_t_id points to JS wrapper<br/>both stored via HCS-1
```

### Step 2: Create UI Block

```mermaid
sequenceDiagram
    participant Developer as Developer
    participant HCS1 as HCS-1 Storage

    Developer->>HCS1: Store HTML Template
    Note over HCS1: HTML template with<br/>action placeholders
    HCS1->>Developer: Template topic ID: 0.0.234567

    Developer->>HCS1: Store Block Definition
    Note over HCS1: {<br/>  "name": "hashlink/nft-form",<br/>  "title": "NFT Form",<br/>  "template_t_id": "0.0.234567",<br/>  "attributes": {...}<br/>}
    HCS1->>Developer: Block topic ID: 0.0.234568
```

### Step 3: Create Assembly

```mermaid
sequenceDiagram
    participant Developer as Developer
    participant AssemblyTopic as Assembly Topic

    Developer->>Hedera: Create Assembly Topic
    Note over Hedera: memo: hcs-12:0:60:2

    Developer->>AssemblyTopic: Register Assembly
    Note over AssemblyTopic: {<br/>  "p": "hcs-12",<br/>  "op": "register",<br/>  "name": "NFT Minting Experience",<br/>  "version": "1.0.0"<br/>}

    Developer->>AssemblyTopic: Add Action
    Note over AssemblyTopic: {<br/>  "p": "hcs-12",<br/>  "op": "add-action",<br/>  "t_id": "0.0.123456",<br/>  "alias": "mint-nft"<br/>}

    Developer->>AssemblyTopic: Add Block
    Note over AssemblyTopic: {<br/>  "p": "hcs-12",<br/>  "op": "add-block",<br/>  "block_t_id": "0.0.234568",<br/>  "actions": {<br/>    "mint": "0.0.123456"<br/>  }<br/>}
```

### Step 4: Register in Global Directory

```mermaid
sequenceDiagram
    participant Developer as Developer
    participant HashLinksRegistry as HashLinks Registry

    Developer->>Hedera: Create HashLinks Registry Topic
    Note over Hedera: memo: hcs-12:1:60:3

    Developer->>HashLinksRegistry: Register HashLink
    Note over HashLinksRegistry: {<br/>  "p": "hcs-12",<br/>  "op": "register",<br/>  "t_id": "0.0.123456",<br/>  "name": "My HashLink Experience",<br/>  "tags": ["nft", "crypto"]<br/>}
```

### Step 5: User Interaction Flow

```mermaid
sequenceDiagram
    participant User as End User
    participant Client as HashLinks Client
    participant HashLinksReg as HashLinks Registry
    participant Assembly as Assembly Registry
    participant Block as Block Registry
    participant Action as Action Registry
    participant Hedera as Hedera

    User->>Client: Access HashLinks Directory
    Client->>HashLinksReg: Load available HashLinks
    HashLinksReg->>Client: Return HashLink directory

    User->>Client: Select HashLink
    Client->>Assembly: Load all messages from assembly topic
    Assembly->>Client: Return operation history

    Client->>Client: Process operations to build state

    Client->>Client: Load block definitions from HCS-1
    Client->>Client: Load block templates from HCS-1

    Client->>Action: Load referenced actions (latest versions)
    Action->>Client: Return WASM modules

    Client->>Client: Render UI with blocks (using HCS-3)
    User->>Client: Interact with UI (e.g., fill form)
    Client->>Client: Validate input

    User->>Client: Submit action
    Client->>Client: Execute WASM module
    Client->>Hedera: Submit transaction
    Hedera->>Client: Confirm transaction
    Client->>User: Display result
```

## Client Architecture

Required components for HashLinks client implementation:

```mermaid
graph LR
    User([User]) --> Client[HashLinks Client]
    
    Client --> RM[Registry<br/>Manager]
    Client --> WE[WASM<br/>Executor]
    Client --> BR[Block<br/>Renderer]
    
    RM --> Registries[HCS Registries]
    WE --> Actions[Action Modules]
    BR --> Blocks[UI Blocks]
    
    Registries --> Network[Hedera Network]
    Actions --> Network
    Blocks --> Network
    
    WE -.-> Wallet[Wallet]
    BR -.-> Editor[Gutenberg Editor]
    
    style User fill:#f9d5e5,stroke:#333,stroke-width:2px
    style Client fill:#d5e5f9,stroke:#333,stroke-width:2px
    style Network fill:#e6f5e6,stroke:#333,stroke-width:2px
```


## HCS-10 Interoperability

HashLinks interoperate with the HCS-10 OpenConvAI standard for agent-to-agent and human–agent interactions.

### Agent Integration

Agents can embed HashLinks in their communications to provide interactive experiences:

```json
{
  "p": "hcs-10",
  "op": "message",
  "operator_id": "0.0.789101@0.0.123456",
  "data": "{\"content\":\"Here's a visualization of the data you requested\",\"hashlink\":\"hcs://15/0.0.567890\"}",
  "m": "Message with embedded HashLink"
}
```

Example: DeFi loan coordination between agents:

```mermaid
sequenceDiagram
    participant LenderAgent as Lending Agent
    participant ConnectionTopic as Connection Topic
    participant BorrowerAgent as Borrower Agent
    participant Hedera as Hedera

    BorrowerAgent->>ConnectionTopic: Request loan via HashLink
    ConnectionTopic->>LenderAgent: Deliver request

    LenderAgent->>LenderAgent: GET() to retrieve on-chain data

    LenderAgent->>ConnectionTopic: Share lending parameters
    ConnectionTopic->>BorrowerAgent: Deliver parameters

    BorrowerAgent->>BorrowerAgent: POST() to generate deposit tx bytes
    BorrowerAgent->>ConnectionTopic: Record tx parameters & bytes

    BorrowerAgent->>Hedera: Submit transaction
    Hedera->>BorrowerAgent: Return receipt

    BorrowerAgent->>ConnectionTopic: Send transaction receipt
    ConnectionTopic->>LenderAgent: Deliver receipt

    LenderAgent->>LenderAgent: POST() to generate borrow tx bytes
    LenderAgent->>Hedera: Submit transaction

    LenderAgent->>ConnectionTopic: Record verified transaction

    Note over LenderAgent,BorrowerAgent: Both agents use HashLinks to<br/>verify and monitor the loan
```

### Human–Agent Interaction

HashLinks enable agents to provide interactive interfaces rather than text-based instructions:

```mermaid
sequenceDiagram
    participant Human as Human User
    participant Agent as Agent
    participant HashLink as HashLink Experience

    Human->>Agent: "Help me mint an NFT"
    Agent->>Agent: Generate appropriate HashLink
    Agent->>Human: Provide HashLink for NFT minting
    Human->>HashLink: Interact with NFT form
    HashLink->>Agent: Submit form data
    Agent->>Agent: Process with WASM module
    Agent->>Human: Confirm NFT minted with receipt
```

### Integration Architecture

```mermaid
graph LR
    %% Define main entities
    AgentNode([Agent])
    Human([Human User])

    %% Define key components in layers with shorter labels
    subgraph " "
        direction TB
        subgraph HCS10["HCS-10 Layer"]
            direction LR
            CT[Connection<br/>Topic]
            IT[Inbound<br/>Topic]
            OT[Outbound<br/>Topic]
            AR[Agent<br/>Registry]
        end

        subgraph HCS15["hcs-12 Layer"]
            direction LR
            HL[HashLink<br/>Experience]
            AL[Actions]
            BL[Blocks]
            AsmL[Assembly]
        end
    end

    %% Define connections
    AgentNode --> HCS10
    Human --> HCS10
    AgentNode --> HCS15
    Human --> HCS15
    HCS10 -.-> HCS15

    %% Styling
    style AgentNode fill:#f9d5e5,stroke:#333,stroke-width:2px
    style Human fill:#f9d5e5,stroke:#333,stroke-width:2px
    style CT fill:#e6ccff,stroke:#333
    style IT fill:#e6ccff,stroke:#333
    style OT fill:#e6ccff,stroke:#333
    style AR fill:#e6ccff,stroke:#333
    style HL fill:#f5f5f5,stroke:#333,stroke-width:3px
    style AL fill:#d5f9e5,stroke:#333
    style BL fill:#d5f9e5,stroke:#333
    style AsmL fill:#d5f9e5,stroke:#333
    style HCS10 fill:#f0e6ff,stroke:#333,stroke-width:2px
    style HCS15 fill:#e6f5e6,stroke:#333,stroke-width:2px
```

### Technical Implementation

1. **Message Extension**: Add a `hashlink` field to HCS-10 messages for HashLink references
2. **Context Passing**: WASM modules receive conversation context via the memo parameter

## Compatibility

This standard is compatible with:
- HCS-1 for file storage
- HCS-2 for topic registries  
- HCS-3 for resource loading
- HCS-11 for agent profiles
- WordPress Gutenberg block editor

## References

1. [HCS-1 Standard: File Data Management](../hcs-1.md)
2. [HCS-2 Standard: Advanced Topic Registries](../hcs-2.md)
3. [HCS-3 Standard: Recursion within Hedera Consensus Service](../hcs-3.md)
4. [WordPress Gutenberg Block API](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-registration/)
