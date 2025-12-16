# Actions Layer - WASM Modules

## Overview

For implementation details of the Actions Layer in the standards-sdk, see the [HCS-12 SDK Guide](/docs/libraries/standards-sdk/hcs-12/) documentation.

The Actions Layer provides deterministic computation through WebAssembly modules. By compiling Rust to WASM and executing in a sandboxed environment, Actions enable deterministic computation and transaction preparation.

## WASM Module Interface

Every WASM module MUST implement the `WasmInterface` with three core methods:

```typescript
interface WasmInterface {
  // Returns a deterministic JSON string containing module metadata
  INFO(): string;

  // Process action with parameters
  POST(
    action: string,
    params: string,
    network: 'mainnet' | 'testnet',
    hashLinkMemo: string
  ): Promise<string>;

  // Retrieve information about action
  GET(
    action: string,
    params: string,
    network: 'mainnet' | 'testnet'
  ): Promise<string>;
}
```

### Method Specifications

#### INFO Method

The INFO method provides metadata about the module's capabilities. This method MUST:

- Be deterministic (always return the same output)
- Return valid JSON
- Include all required fields
- Be callable without any parameters

```typescript
type ModuleInfo = {
  name: string;
  version: string;
  hashlinks_version: string;
  creator: string;
  purpose: string;
  actions: Array<ActionDefinition>;
  capabilities: Array<Capability>;
  plugins: Array<PluginDefinition>;
};
```

#### POST Method

The POST method executes actions that modify state or submit transactions. Parameters:

- `action`: The specific action to execute (MUST match one from INFO)
- `params`: JSON string containing action parameters
- `network`: Target network for the operation
- `hashLinkMemo`: Context from the HashLink assembly

#### GET Method

The GET method retrieves information without modifying state. It follows the same parameter structure as POST but MUST NOT submit transactions or modify external state.

## Action Definitions

Each action in a module MUST be fully defined:

```typescript
type ActionDefinition = {
  name: string;
  description: string;
  inputs: Array<ParameterDefinition>;
  outputs: Array<ParameterDefinition>;
  required_capabilities: Array<Capability>;
};

type ParameterDefinition = {
  name: string;
  param_type:
    | 'string'
    | 'number'
    | 'boolean'
    | 'array'
    | 'object'
    | 'network'
    | 'address'
    | 'bigint';
  description: string;
  required: boolean;
  validation?: ValidationRule;
};

type ValidationRule = {
  // String validations (matches z.string() methods)
  regex?: string; // z.string().regex()
  min?: number; // z.string().min() - minimum length
  max?: number; // z.string().max() - maximum length
  length?: number; // z.string().length() - exact length
  email?: boolean; // z.string().email()
  url?: boolean; // z.string().url()
  uuid?: boolean; // z.string().uuid()
  cuid?: boolean; // z.string().cuid()
  cuid2?: boolean; // z.string().cuid2()
  ulid?: boolean; // z.string().ulid()
  datetime?: boolean; // z.string().datetime()
  ip?: boolean; // z.string().ip()
  startsWith?: string; // z.string().startsWith()
  endsWith?: string; // z.string().endsWith()
  includes?: string; // z.string().includes()

  // Number validations (matches z.number() methods)
  gt?: number; // z.number().gt()
  gte?: number; // z.number().gte()
  lt?: number; // z.number().lt()
  lte?: number; // z.number().lte()
  int?: boolean; // z.number().int()
  positive?: boolean; // z.number().positive()
  nonnegative?: boolean; // z.number().nonnegative()
  negative?: boolean; // z.number().negative()
  nonpositive?: boolean; // z.number().nonpositive()
  multipleOf?: number; // z.number().multipleOf()
  finite?: boolean; // z.number().finite()
  safe?: boolean; // z.number().safe()

  // Array validations (matches z.array() methods)
  nonempty?: boolean; // z.array().nonempty()
  // min/max already defined above work for arrays too

  // General validations
  literal?: string | number | boolean; // z.literal()
  enum?: Array<string>; // z.enum()
  nullable?: boolean; // .nullable()
  nullish?: boolean; // .nullish()
  optional?: boolean; // .optional()

  // For arrays and objects
  element?: ParameterDefinition; // For arrays: element schema
  shape?: Record<string, ParameterDefinition>; // For objects: property schemas
  strict?: boolean; // For objects: z.object().strict()
  passthrough?: boolean; // For objects: z.object().passthrough()
  catchall?: ParameterDefinition; // For objects: z.object().catchall()
};
```

## Capabilities System

Capabilities define what resources and permissions a module requires:

```typescript
type Capability = {
  type: 'network' | 'transaction' | 'storage' | 'external_api';
  value:
    | NetworkCapability
    | TransactionCapability
    | StorageCapability
    | ExternalApiCapability;
};

type NetworkCapability = {
  networks: Array<'mainnet' | 'testnet'>;
  operations: Array<'query' | 'submit'>;
};

type TransactionCapability = {
  transaction_types: Array<
    'token_transfer' | 'token_create' | 'token_mint' | 'contract_call'
  >;
  max_fee_hbar?: number;
};

type StorageCapability = {
  storage_types: Array<'hcs' | 'ipfs' | 'arweave'>;
  max_size_bytes?: number;
};

type ExternalApiCapability = {
  allowed_domains: string[];
  rate_limit?: number;
};
```

## Registration Process

### 1. Create Registry Topic

Create a topic with the appropriate memo format:

```
hcs-12:1:60:0
```

Where:

- `1` = Non-indexed (only latest message matters)
- `60` = 60-second TTL for caching
- `0` = Action registry type

### 2. Register Module

Submit the registration message to the Action Registry:

```json
{
  "p": "hcs-12",
  "op": "register",
  "t_id": "0.0.123456",
  "hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "wasm_hash": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2",
  "info_t_id": "0.0.456789",
  "m": "NFT minting module v1.0.0"
}
```

Where:

- `t_id`: Topic ID where the WASM binary is stored (via [HCS-1](../hcs-1.md))
- `hash`: SHA-256 hash of the module's INFO method output
- `wasm_hash`: SHA-256 hash of the WASM binary
- `info_t_id`: (Optional) Topic ID containing the INFO method output - useful when INFO output exceeds 1KB

## Hash Verification

Clients must verify module integrity before execution:

### 1. Metadata Hash Verification

```javascript
// Load and execute INFO method
const infoJson = await wasmModule.INFO();
const computedHash = sha256(infoJson);

// Compare with registered hash
if (computedHash !== registeredHash) {
  throw new Error('Module metadata tampered');
}
```

### 2. Binary Hash Verification

```javascript
// Compute hash of WASM binary
const binaryHash = sha256(wasmBinary);

// Compare with registered hash
if (binaryHash !== registeredWasmHash) {
  throw new Error('Module binary tampered');
}
```

## Security Considerations

### 1. Sandboxing

WASM modules must run in a sandboxed environment with:

- Memory isolation
- CPU time limits
- Network access restrictions
- File system isolation

### 2. Parameter Validation

All input parameters must be validated before processing using schema validation libraries like Zod:

```typescript
import { z } from 'zod';

// Define parameter schemas based on the action definition
function createParameterSchema(definition: ParameterDefinition): z.ZodTypeAny {
  let schema: z.ZodTypeAny;
  const v = definition.validation;

  // Base type validation
  switch (definition.param_type) {
    case 'string':
      schema = z.string();
      if (v?.min !== undefined) {
        schema = schema.min(v.min);
      }
      if (v?.max !== undefined) {
        schema = schema.max(v.max);
      }
      if (v?.length !== undefined) {
        schema = schema.length(v.length);
      }
      if (v?.regex) {
        schema = schema.regex(new RegExp(v.regex));
      }
      if (v?.email) {
        schema = schema.email();
      }
      if (v?.url) {
        schema = schema.url();
      }
      if (v?.uuid) {
        schema = schema.uuid();
      }
      if (v?.cuid) {
        schema = schema.cuid();
      }
      if (v?.cuid2) {
        schema = schema.cuid2();
      }
      if (v?.ulid) {
        schema = schema.ulid();
      }
      if (v?.datetime) {
        schema = schema.datetime();
      }
      if (v?.ip) {
        schema = schema.ip();
      }
      if (v?.startsWith) {
        schema = schema.startsWith(v.startsWith);
      }
      if (v?.endsWith) {
        schema = schema.endsWith(v.endsWith);
      }
      if (v?.includes) {
        schema = schema.includes(v.includes);
      }
      break;

    case 'number':
      schema = z.number();
      if (v?.gt !== undefined) {
        schema = schema.gt(v.gt);
      }
      if (v?.gte !== undefined) {
        schema = schema.gte(v.gte);
      }
      if (v?.lt !== undefined) {
        schema = schema.lt(v.lt);
      }
      if (v?.lte !== undefined) {
        schema = schema.lte(v.lte);
      }
      if (v?.int) {
        schema = schema.int();
      }
      if (v?.positive) {
        schema = schema.positive();
      }
      if (v?.nonnegative) {
        schema = schema.nonnegative();
      }
      if (v?.negative) {
        schema = schema.negative();
      }
      if (v?.nonpositive) {
        schema = schema.nonpositive();
      }
      if (v?.multipleOf) {
        schema = schema.multipleOf(v.multipleOf);
      }
      if (v?.finite) {
        schema = schema.finite();
      }
      if (v?.safe) {
        schema = schema.safe();
      }
      break;

    case 'bigint':
      schema = z.bigint();
      break;

    case 'boolean':
      schema = z.boolean();
      break;

    case 'array':
      schema = v?.element
        ? z.array(createParameterSchema(v.element))
        : z.array(z.unknown());
      if (v?.min !== undefined) {
        schema = schema.min(v.min);
      }
      if (v?.max !== undefined) {
        schema = schema.max(v.max);
      }
      if (v?.nonempty) {
        schema = schema.nonempty();
      }
      break;

    case 'object':
      if (v?.shape) {
        const shape = Object.entries(v.shape).reduce((acc, [key, def]) => {
          acc[key] = createParameterSchema(def);
          return acc;
        }, {} as Record<string, z.ZodTypeAny>);

        if (v?.strict) {
          schema = z.object(shape).strict();
        } else if (v?.passthrough) {
          schema = z.object(shape).passthrough();
        } else {
          schema = z.object(shape);
        }

        if (v?.catchall) {
          schema = schema.catchall(createParameterSchema(v.catchall));
        }
      } else {
        schema = z.object({}).passthrough();
      }
      break;

    case 'address':
      schema = z.string().regex(/^0\.0\.\d+$/, 'Invalid Hedera address format');
      break;

    case 'network':
      schema = z.enum(['mainnet', 'testnet']);
      break;

    default:
      schema = z.unknown();
  }

  // Apply general validations
  if (v?.literal !== undefined) {
    schema = z.literal(v.literal);
  }

  if (v?.enum) {
    schema = z.enum(v.enum as [string, ...string[]]);
  }

  // Apply modifiers (order matters in Zod)
  if (v?.nullable) {
    schema = schema.nullable();
  }

  if (v?.nullish) {
    schema = schema.nullish();
  }

  // Check both v?.optional and definition.required
  if (v?.optional || !definition.required) {
    schema = schema.optional();
  }

  return schema;
}

// Validate action parameters
function validateActionParams(action: ActionDefinition, params: unknown) {
  const schema = z.object(
    action.inputs.reduce((acc, input) => {
      acc[input.name] = createParameterSchema(input);
      return acc;
    }, {} as Record<string, z.ZodTypeAny>)
  );

  return schema.parse(params);
}
```

### 3. Resource Limits

Implementations MUST enforce resource limits. Recommended defaults:

- Execution time SHOULD NOT exceed 30 seconds
- Memory usage SHOULD NOT exceed 256MB
- Output size SHOULD NOT exceed 1MB
- Apply rate limiting per module

## Example Implementation

### HashLink Donation & NFT Purchase Module

```rust
use wasm_bindgen::prelude::*;
use hashlinks_capabilities::prelude::*;
use serde_json::json;

#[wasm_bindgen]
pub struct WasmInterface;

#[wasm_bindgen]
impl WasmInterface {
    #[wasm_bindgen(constructor)]
    pub fn new() -> Self {
        Self
    }

    #[wasm_bindgen]
    pub fn INFO(&self) -> Result<String, JsValue> {
        let info = ModuleInfo::builder()
            .name("Example Module")
            .version("0.1.0")
            .hashlinks_version("0.1.0")
            .creator("example")
            .purpose("Example donation and NFT purchase operations")
            .add_capability(Transaction::CryptoTransfer.into())
            .add_capability(Transaction::TokenTransfer.into())
            .add_capability(Network::Mainnet.into())
            .add_capability(Network::Testnet.into())
            .add_plugin(PluginRequirement::new(
                "example-plugin",
                "1.0.0",
                "https://example.org/wasm-plugin",
                "Example marketplace integration"
            ))
            .add_action(
                ActionDefinition::new("donate", "Donate HBAR")
                    .input("amount", ParamType::Number, "Amount in HBAR", true)
                    .input("sender", ParamType::Address, "Sender address", true)
                    .input("network", ParamType::NetworkId, "Network to execute on", true)
                    .output("transaction_bytes", ParamType::Object, "Transaction data", true)
                    .output("success", ParamType::Boolean, "Operation success", true)
                    .requires_capability(Transaction::CryptoTransfer.into())
            )
            .add_action(
                ActionDefinition::new("nft-purchase", "Purchase NFT")
                    .input("token_id", ParamType::String, "Token ID", true)
                    .input("serial_number", ParamType::Number, "Serial number", true)
                    .input("price", ParamType::Number, "Price in HBAR", true)
                    .input("sender", ParamType::Address, "Buyer address", true)
                    .output("transaction_bytes", ParamType::Object, "Transaction data", true)
                    .output("success", ParamType::Boolean, "Operation success", true)
                    .requires_capability(Transaction::TokenTransfer.into())
            )
            .build();

        Ok(serde_json::to_string(&info).unwrap())
    }

    #[wasm_bindgen]
    pub async fn POST(
        &self,
        action: &str,
        params_str: &str,
        network: &str,
        hash_link_memo: &str,
    ) -> Result<String, JsValue> {
        match action {
            "donate" => {
                let params: DonationParams = serde_json::from_str(params_str)
                    .map_err(|e| JsValue::from_str(&e.to_string()))?;

                // Validate amount
                if params.amount <= 0.0 {
                    return Ok(json!({
                        "success": false,
                        "error": "Invalid amount: must be positive"
                    }).to_string());
                }

                // Create transfer transaction
                let hedera_client = HederaClient::new()?;
                let recipient_id = if network == "mainnet" {
                    "0.0.800"
                } else {
                    "0.0.800"
                };

                let tx_bytes = hedera_client.create_transfer(
                    params.sender.clone(),
                    recipient_id.to_string(),
                    (params.amount * 100_000_000.0) as i64, // Convert HBAR to tinybars
                    hash_link_memo.to_string(),
                )?;

                Ok(json!({
                    "success": true,
                    "data": {
                        "transaction_bytes": {
                            "type": "transaction",
                            "data": tx_bytes
                        }
                    },
                    "message": "Transaction created successfully"
                }).to_string())
            }

            "nft-purchase" => {
                let params: NFTPurchaseParams = serde_json::from_str(params_str)
                    .map_err(|e| JsValue::from_str(&e.to_string()))?;

                // Call external service to create NFT purchase transaction
                let service = MarketplaceService::new(&params.api_key);
                let response = service.purchase_nft(
                    &params.token_id,
                    params.serial_number,
                    &params.sender,
                    params.price,
                ).await?;

                Ok(json!({
                    "success": true,
                    "data": {
                        "transaction_bytes": response.transaction_bytes
                    },
                    "message": "NFT purchase transaction created"
                }).to_string())
            }

            _ => Ok(json!({
                "success": false,
                "error": format!("Unknown action: {}", action)
            }).to_string())
        }
    }

    #[wasm_bindgen]
    pub async fn GET(
        &self,
        action: &str,
        params_str: &str,
        network: &str
    ) -> Result<String, JsValue> {
        match action {
            "donate" => {
                // Return donation form configuration
                Ok(json!({
                    "image": "https://example.org/donate.png",
                    "title": "Donation",
                    "description": format!("Donation parameters for {}", network),
                    "label": "Donate Now",
                    "parameters": [
                        {
                            "type": "hbar",
                            "name": "amount",
                            "label": "Amount to donate (HBAR)",
                            "required": true,
                            "min": 1,
                            "preset_values": [10, 50, 100],
                            "custom_value": true
                        }
                    ]
                }).to_string())
            }

            "nft-purchase" => {
                let params: serde_json::Value = serde_json::from_str(params_str)?;
                let token_id = params.get("tokenId").and_then(|v| v.as_str()).unwrap_or("");

                // Fetch available NFT
                let service = MarketplaceService::new("");
                let nft = service.get_random_nft(token_id, None).await?;

                Ok(json!({
                    "image": nft.image,
                    "title": nft.name,
                    "description": "Purchase this NFT",
                    "label": format!("Buy Now ({}ℏ)", nft.price),
                    "parameters": [
                        {
                            "type": "number",
                            "name": "serial_number",
                            "value": nft.serial_number,
                            "hidden": true
                        },
                        {
                            "type": "hbar",
                            "name": "price",
                            "value": nft.price,
                            "hidden": true
                        }
                    ]
                }).to_string())
            }

            _ => Ok(json!({
                "error": format!("Unknown action: {}", action)
            }).to_string())
        }
    }
}
```

## Advanced Features

### 1. Plugin System

Modules can declare dependencies on external plugins:

```typescript
type PluginDefinition = {
  name: string;
  version: string;
  url: string;
  description: string;
  required: boolean;
};
```

### 2. Composable Actions

Actions can be composed by referencing other modules:

```json
{
  "name": "complex-defi",
  "actions": [
    {
      "name": "swap-and-stake",
      "description": "Swap tokens and stake in one action",
      "composed_actions": [
        {
          "module": "0.0.123456",
          "action": "swap"
        },
        {
          "module": "0.0.789012",
          "action": "stake"
        }
      ]
    }
  ]
}
```

### 3. State Management

Modules can maintain internal state between calls:

```rust
static mut STATE: Option<HashMap<String, String>> = None;

#[wasm_bindgen]
pub fn init_state() {
    unsafe {
        STATE = Some(HashMap::new());
    }
}
```

## Best Practices

1. **Deterministic Execution**: Ensure all operations are deterministic
2. **Error Handling**: Provide clear error messages with actionable information
3. **Gas Optimization**: Minimize computation and memory usage
4. **Version Management**: Use semantic versioning and maintain backwards compatibility
5. **Documentation**: Provide comprehensive documentation for each action
6. **Testing**: Include comprehensive test suites with edge cases
7. **Audit Trail**: Log all operations for debugging and auditing

## Code Verification

### Binary Verification

Similar to Solidity contract verification, users can verify WASM modules:

1. **Fetch the WASM binary** from the `t_id` using HCS-1
2. **Compute SHA-256 hash** of the downloaded binary
3. **Compare with `wasm_hash`** in the registry

```typescript
async function verifyWasmModule(
  AssemblyRegistryEntry: AssemblyRegistryEntry
): Promise<boolean> {
  // Fetch WASM binary from HCS-1
  const wasmBinary = await fetchFromHCS1(AssemblyRegistryEntry.t_id);

  // Compute hash
  const computedHash = sha256(wasmBinary);

  // Verify against registry
  return computedHash === AssemblyRegistryEntry.wasm_hash;
}
```

### Metadata Verification

Verify the module's declared capabilities:

```typescript
async function verifyModuleInfo(
  wasmModule: WebAssembly.Module
): Promise<boolean> {
  // Instantiate and call INFO
  const instance = await WebAssembly.instantiate(wasmModule);
  const infoJson = instance.exports.INFO();

  // Compute hash of INFO output
  const computedHash = sha256(infoJson);

  // Verify against registry
  return computedHash === AssemblyRegistryEntry.hash;
}
```

### Source Code Verification

For full transparency, developers can publish source code:

```typescript
interface SourceCodeVerification {
  source_t_id: string; // HCS-1 topic containing source archive
  source_hash: string; // SHA-256 hash of the source archive file
  compiler_version: string; // Exact rustc version (e.g., "1.75.0")
  cargo_version: string; // Exact cargo version
  target: string; // Target triple (e.g., "wasm32-unknown-unknown")
  profile: string; // Build profile ("release" or custom)
  build_flags: string[]; // Additional cargo/rustc flags
  lockfile_hash: string; // SHA-256 of Cargo.lock
  source_structure: SourceStructure;
}

interface SourceStructure {
  format: 'tar.gz' | 'zip' | 'car'; // Archive format
  root_manifest: string; // Path to Cargo.toml (e.g., "./Cargo.toml")
  includes_lockfile: boolean; // Whether Cargo.lock is included
  workspace_members?: string[]; // For workspace projects
}
```

#### Reproducible Build Requirements

For deterministic WASM compilation, the source archive MUST include:

1. **Complete source tree** - All `.rs` files and subdirectories
2. **Cargo.toml** - With exact dependency versions (no wildcards)
3. **Cargo.lock** - Pins all transitive dependencies
4. **Build configuration** - Any `.cargo/config.toml` files
5. **Feature flags** - Document which features were enabled

#### Archive Structure Example

```
wasm-module-v1.0.0.tar.gz
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── lib.rs
│   ├── types.rs
│   ├── hedera.rs
│   └── modules/
│       ├── mod.rs
│       └── nft.rs
├── .cargo/
│   └── config.toml
└── rust-toolchain.toml  # Pins exact Rust version
```

#### Verification Process

Anyone can verify the source code in two steps:

**Step 1: Verify Source Archive Integrity**

```bash
# Download source archive from HCS-1
hcs1-fetch 0.0.345678 > source.tar.gz

# Verify the source archive hash matches what's registered
sha256sum source.tar.gz
# Should match the source_hash in the registry entry
```

**Step 2: Reproduce the Build**

```bash
# Extract source
tar -xzf source.tar.gz

# Install exact toolchain
rustup toolchain install 1.75.0
rustup target add wasm32-unknown-unknown

# Build with exact flags
cargo build --release --target wasm32-unknown-unknown \
  --locked \
  --features "hedera"

# Compare output hash with registered wasm_hash
sha256sum target/wasm32-unknown-unknown/release/module.wasm
```

#### Determinism Challenges

Several factors can affect reproducibility:

1. **Timestamps** - Use `--remap-path-prefix` to remove
2. **Random seeds** - Ensure build doesn't use randomness
3. **Host dependencies** - System libraries can affect output
4. **Compiler optimizations** - Must use exact same settings

Best practice is to provide a Docker image:

```dockerfile
FROM rust:1.75.0
WORKDIR /build
COPY . .
RUN cargo build --release --target wasm32-unknown-unknown --locked
```

This approach supports reproducible builds and verifiable artifacts with complete on-chain storage.

## Migration and Updates

To update a module:

1. Deploy new WASM binary using HCS-1
2. Submit new registration with updated hashes
3. Clients automatically use the latest version
4. Old versions remain accessible via historical messages

```json
{
  "p": "hcs-12",
  "op": "register",
  "t_id": "0.0.234567",
  "hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "wasm_hash": "d2d2d2d2e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2",
  "source_verification": {
    "source_t_id": "0.0.345678",
    "source_hash": "f1f1f1f1e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2",
    "compiler_version": "1.75.0",
    "cargo_version": "1.75.0",
    "target": "wasm32-unknown-unknown",
    "profile": "release",
    "build_flags": ["--locked", "--features", "hedera"],
    "lockfile_hash": "c3c3c3c3e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2",
    "source_structure": {
      "format": "tar.gz",
      "root_manifest": "./Cargo.toml",
      "includes_lockfile": true
    }
  },
  "previous_version": "1.0.0",
  "migration_notes": "Added support for batch minting",
  "m": "NFT minting module v2.0.0"
}
```
