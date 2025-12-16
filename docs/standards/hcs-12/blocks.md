# Blocks Layer - UI Components

## Overview

For implementation details of the Blocks Layer in the standards-sdk, see the [HCS-12 SDK Guide](/docs/libraries/standards-sdk/hcs-12/) documentation.

The Blocks Layer implements a decentralized, composable UI component system based on WordPress's Gutenberg block format. Blocks are stored as immutable assets via HCS-1, with their definitions and templates stored separately. Assemblies reference blocks directly by their HCS-1 topic IDs.

## Block Architecture

### Gutenberg Foundation

HashLinks adopts the Gutenberg block format for its proven flexibility and extensive ecosystem. Key benefits include:

1. **Composability**: Blocks can contain other blocks, enabling complex layouts
2. **Portability**: Standardized format works across different implementations
3. **Extensibility**: Rich attribute system for customization
4. **Accessibility**: Built-in support for screen readers and keyboard navigation

### Block Storage Structure

Blocks consist of two parts stored via HCS-1:

1. **Block Template** (HTML): The rendering template with placeholders
2. **Block Definition** (JSON): The Gutenberg block metadata including template reference

The template is stored first, then referenced in the block definition:

```json
{
  "apiVersion": 3,
  "name": "hashlink/token-transfer",
  "title": "Token Transfer",
  "category": "hashlink/actions",
  "icon": "money",
  "description": "Transfer tokens between accounts",
  "keywords": ["token", "transfer", "payment"],
  "template_t_id": "0.0.123456",
  "attributes": {
    "tokenId": {
      "type": "string",
      "default": ""
    },
    "amount": {
      "type": "number",
      "default": 1
    }
  },
  "supports": {
    "align": ["wide", "full"],
    "anchor": true
  }
}
```

This approach keeps the template separate (avoiding JSON escaping issues) while making blocks self-contained.

## Gutenberg Compatibility

HashLinks blocks are designed to be fully compatible with the WordPress Gutenberg editor, enabling:

1. **Drag-and-Drop Editing**: Users can visually arrange blocks using any Gutenberg-compatible editor
2. **Visual Block Selection**: Browse and insert blocks from a visual library
3. **Live Preview**: See changes in real-time as blocks are configured
4. **Nested Composition**: Create complex layouts with container blocks
5. **Responsive Design**: Blocks automatically adapt to different screen sizes

### Using Gutenberg Editors

HashLinks blocks can be created and edited using any Gutenberg-compatible editor:

1. **Open Editor**: Use WordPress, Gutenberg standalone, or any compatible block editor
2. **Insert Blocks**: Browse the HashLink category in the block inserter
3. **Configure**: Use the block settings panel to configure attributes
4. **Preview**: See real-time preview of your HashLink experience
5. **Export**: Save the block structure as JSON for registration

## Block Creation Process

Blocks are created by storing their template first, then their definition:

### Step 1: Store Block Template

Store the HTML template via HCS-1:

```html
<!-- token-transfer-template.html -->
<div class="hashlink-block token-transfer" data-block-id="{{blockId}}">
  <h3>{{attributes.title}}</h3>
  
  <form data-action="{{actionTopicId}}" data-params='{"operation": "transfer"}'>
    <label>
      Token ID:
      <input type="text" name="tokenId" value="{{attributes.tokenId}}" />
    </label>
    
    <label>
      Recipient:
      <input type="text" name="recipient" value="{{attributes.recipient}}" />
    </label>
    
    <label>
      Amount:
      <input type="number" name="amount" value="{{attributes.amount}}" />
    </label>
    
    <button type="submit">Transfer</button>
  </form>
  
  {{#if actionResults.transfer}}
  <div class="result success">
    Transfer complete! TX: {{actionResults.transfer.transactionId}}
  </div>
  {{/if}}
</div>
```

Store this template via HCS-1. The returned topic ID (e.g., "0.0.123456") will be referenced in the block definition.

### Step 2: Store Block Definition

Store the block definition with template reference:

Block definition JSON:

```json
{
  "apiVersion": 3,
  "name": "hashlink/token-transfer",
  "title": "Token Transfer Form",
  "category": "hashlink/actions",
  "description": "Transfer tokens with a customizable form",
  "keywords": ["token", "transfer", "hedera"],
  "template_t_id": "0.0.123456",
  "attributes": {
    "tokenId": {
      "type": "string",
      "default": ""
    },
    "amount": {
      "type": "number", 
      "default": 1
    },
    "recipient": {
      "type": "string",
      "default": ""
    }
  },
  "supports": {
    "align": ["wide", "full"],
    "anchor": true
  }
}
```

Store this JSON via HCS-1. The returned topic ID (e.g., "0.0.123457") is used to reference the block.

### Step 3: Use in Assembly

Reference the block in an assembly by its definition topic ID:

```json
{
  "p": "hcs-12",
  "op": "add-block",
  "block_t_id": "0.0.123457",
  "actions": {
    "transfer": "0.0.789012"
  },
  "attributes": {
    "tokenId": "0.0.456789",
    "amount": 100
  }
}
```

## Template System

### HTML Templates

Block templates use standard HTML with Handlebars-style placeholders and include resources via HCS-3:

```html
<link
  rel="stylesheet"
  data-src="hcs://1/0.0.111111"
  data-script-id="token-transfer-styles"
/>

<div
  class="hashlink-block {{attributes.className}}"
  data-block-id="{{blockId}}"
>
  <h3>{{attributes.title}}</h3>

  {{#if attributes.showDescription}}
  <p>{{attributes.description}}</p>
  {{/if}}

  <form data-action="{{actions.0}}">
    <label>
      Token ID:
      <input type="text" name="tokenId" value="{{attributes.tokenId}}" />
    </label>

    <label>
      Amount:
      <input type="number" name="amount" value="{{attributes.defaultAmount}}" />
    </label>

    {{#if attributes.showAdvanced}}
    <div class="advanced-options">
      <!-- Additional fields -->
    </div>
    {{/if}}

    <button type="submit">Transfer</button>
  </form>

  {{#if action.result}}
  <div class="result">{{ action.result.message }}</div>
  {{/if}}
</div>

<!-- Block-specific JavaScript -->
<script
  data-src="hcs://1/0.0.222222"
  data-script-id="token-transfer-logic"
></script>
```

### Resource Loading with HCS-3

Resources are loaded using the HCS-3 protocol:

```html
<link
  rel="stylesheet"
  data-src="hcs://1/0.0.123456"
  data-script-id="block-styles"
/>

<script
  data-src="hcs://3/0.0.789012"
  data-dependencies="['jquery', 'react']"
></script>

<img
  data-src="hcs://1/{{ attributes.imageTopicId }}"
  alt="{{ attributes.imageAlt }}"
  loading="lazy"
/>
```

## Action Binding

Blocks bind to actions using topic IDs directly. This ensures deterministic behavior and clear dependencies.

### Declaring Actions in Assembly

When adding a block to an assembly, provide a map of action names to topic IDs:

```json
{
  "p": "hcs-12",
  "op": "add-block",
  "block_t_id": "0.0.123456",
  "actions": {
    "primary": "0.0.789012"
  },
  "attributes": {
    "title": "Transfer HBAR"
  }
}
```

### Using Actions in Templates

Templates receive actions as a map of names to topic IDs:

```handlebars
<button data-action="{{actions.primary}}">Submit Payment</button>

<form data-action="{{actions.primary}}" data-params='{"operation": "transfer"}'>
  <input type="text" name="recipient" />
  <input type="number" name="amount" />
  <button type="submit">Transfer</button>
</form>

<div data-action-click="{{actions.primary}}" data-params='{"amount": 100, "recipient": "0.0.123456"}'>
  Quick Transfer 100 HBAR
</div>
```

For blocks with multiple actions:

```json
{
  "p": "hcs-12",
  "op": "add-block",
  "block_t_id": "0.0.123456",
  "actions": {
    "transfer": "0.0.789012",
    "validate": "0.0.789013",
    "getBalance": "0.0.789014"
  }
}
```

In the template:

```handlebars
<form data-action="{{actions.transfer}}" data-validate="{{actions.validate}}">
  <button type="button" data-action-click="{{actions.getBalance}}">Check Balance</button>
  <input type="text" name="recipient" />
  <button type="submit">Transfer</button>
</form>
```

### Action Execution Flow

1. **User Interaction**: User clicks button or submits form
2. **Data Collection**: Form data or data-params are collected
3. **Action Loading**: The WASM module is loaded from the action topic ID
4. **WASM Execution**: The action's POST method is called with:
   - `action`: The method name from the action definition
   - `params`: Collected form data or specified parameters
   - `network`: Current Hedera network
   - `hashLinkMemo`: Assembly identifier
5. **Result Handling**: The WASM response updates the block state

### Event Handlers

Blocks can specify different events for actions:

```handlebars
<button data-action-click="{{actions.primary}}">Click Me</button>

<select data-action-change="{{actions.validate}}" name="token">
  <option value="HBAR">HBAR</option>
  <option value="USDC">USDC</option>
</select>

<div data-action-mouseover="{{actions.preview}}" 
     data-action-params='{"hover": true}'>
  Hover for preview
</div>
```

### Passing Parameters

Parameters can be passed to actions in several ways:

```handlebars
<form data-action="{{actions.transfer}}">
  <input name="recipient" type="text" />
  <input name="amount" type="number" />
  <button type="submit">Send</button>
</form>

<button data-action="{{actions.transfer}}" 
        data-params='{"amount": 50, "token": "HBAR"}'>
  Send 50 HBAR
</button>

<button data-action="{{actions.transfer}}" 
        data-params='{"tokenId": "{{attributes.tokenId}}"}'>
  Transfer {{attributes.tokenName}}
</button>
```

### Action Results

Action results are available for template updates:

```handlebars
{{#if actionResults.transfer}}
  <div class="success">
    Transfer complete! 
    TX: {{actionResults.transfer.transactionId}}
  </div>
{{/if}}

{{#if actionErrors.transfer}}
  <div class="error">
    Transfer failed: {{actionErrors.transfer.message}}
  </div>
{{/if}}
```

## Nested Blocks (Recursive Composition)

HashLinks supports nested blocks through template-based composition. Container blocks can include other blocks directly in their templates, enabling flexible layouts without requiring explicit child references.

### Template-Based Composition

Blocks compose other blocks using HashLink references directly in templates:

```handlebars
<div class="dashboard-container" data-block-id="{{blockId}}">
  <h2>{{attributes.title}}</h2>
  
  <!-- Include other blocks using hcs:// protocol -->
  <div class="stats-section">
    <div data-hashlink="hcs://12/{{attributes.statsBlockId}}" 
         data-attributes='{"theme": "{{attributes.theme}}"}'>
      <!-- Stats block renders here -->
    </div>
  </div>
  
  <!-- Include with HCS-2 non-indexed reference -->
  <div class="actions-section">
    <div data-hashlink="hcs://2/{{attributes.actionsRegistry}}/transfer-form"
         data-attributes='{"tokenId": "{{attributes.defaultToken}}"}'>
      <!-- Transfer form renders here -->
    </div>
  </div>
  
  <!-- Static block reference -->
  <div data-hashlink="hcs://12/0.0.123456">
    <!-- Fixed block always included -->
  </div>
</div>
```

### Dynamic Block Lists

For dynamic lists of blocks, use template helpers:

```handlebars
<div class="portfolio-grid">
  {{#each attributes.portfolioItems}}
    <div class="portfolio-item" 
         data-hashlink="hcs://12/{{this.blockId}}"
         data-attributes='{"tokenId": "{{this.tokenId}}", "amount": "{{this.amount}}"}'>
      <!-- Each portfolio item renders here -->
    </div>
  {{/each}}
</div>
```

### Child Block Context

Child blocks inherit context from their parent:

```handlebars
<!-- Parent block template -->
<div class="parent-block" data-theme="{{attributes.theme}}">
  <h2>{{attributes.title}}</h2>
  
  <!-- Children inherit parent context -->
  <div class="children-wrapper">
    {{{children}}}
  </div>
</div>

<!-- Child block can access parent context -->
<div class="child-block">
  <h3>{{attributes.name}}</h3>
  <!-- Access parent theme via context -->
  <p class="themed-{{parentContext.attributes.theme}}">
    {{attributes.content}}
  </p>
</div>
```

### Recursive Block Example

A dashboard composing nested sections through its template:

```handlebars
<!-- Dashboard Block Template -->
<div class="defi-dashboard">
  <h1>{{attributes.title}}</h1>
  
  <!-- Stats Section (which itself contains nested blocks) -->
  <section class="stats">
    <div data-hashlink="hcs://12/{{attributes.statsContainerId}}"
         data-attributes='{
           "title": "Portfolio Stats",
           "cardRegistry": "{{attributes.statsCardRegistry}}",
           "cardIds": {{json attributes.statCardIds}}
         }'>
    </div>
  </section>
  
  <!-- Actions Section -->
  <section class="actions">
    <div data-hashlink="hcs://12/{{attributes.actionsBlockId}}"
         data-attributes='{
           "defaultToken": "{{attributes.defaultToken}}"
         }'
         data-actions='{
           "transfer": "{{actions.transfer}}",
           "swap": "{{actions.swap}}"
         }'>
    </div>
  </section>
</div>
```

The Stats Container block template can then include its own nested blocks:

```handlebars
<!-- Stats Container Template -->
<div class="stats-container">
  <h2>{{attributes.title}}</h2>
  <div class="stats-grid">
    {{#each attributes.cardIds}}
      <div data-hashlink="hcs://2/{{../attributes.cardRegistry}}/{{this}}"
           class="stat-card">
      </div>
    {{/each}}
  </div>
</div>
```

### State Management for Nested Blocks

Each block maintains its own state, with child states accessible to parents:

```javascript
// Parent block state
{
  "attributes": {
    "title": "Dashboard",
    "theme": "dark"
  },
  "actionResults": {},
  "childStates": {
    "0.0.100001": {
      "attributes": { "value": 1000 },
      "actionResults": {}
    },
    "0.0.100002": {
      "attributes": { "status": "ready" },
      "actionResults": {}
    }
  }
}
```

### Event Bubbling

Events from child blocks can bubble up to parents:

```handlebars
<!-- Parent template -->
<div class="parent" data-block-id="{{blockId}}" 
     data-handle-child-events="true">
  <h2>{{attributes.title}}</h2>
  {{{children}}}
  
  {{#if childEvents.updated}}
    <p>Child {{childEvents.updated.blockId}} was updated</p>
  {{/if}}
</div>
```

## Block Patterns

Pre-configured block arrangements can be registered as patterns:

```json
{
  "p": "hcs-12",
  "op": "pattern",
  "name": "defi-dashboard",
  "title": "DeFi Dashboard",
  "description": "Complete DeFi dashboard layout",
  "categories": ["hashlink/patterns"],
  "content": {
    "name": "hashlink/container",
    "attributes": {
      "layout": "grid",
      "columns": 2
    },
    "innerBlocks": [
      {
        "name": "hashlink/balance-display",
        "attributes": {
          "tokenId": "0.0.123456"
        }
      },
      {
        "name": "hashlink/token-transfer-form",
        "attributes": {
          "tokenId": "0.0.123456"
        }
      }
    ]
  }
}
```


## Security Considerations

### Template Sanitization

All HTML templates MUST be sanitized:

- Remove script tags (use HCS-3 for scripts)
- Sanitize event handlers
- Validate URLs and links
- Escape user-provided content

### Action Validation

Blocks MUST validate action bindings:

- Verify action exists in registry
- Check if block is allowed to use the action
- Validate that required actions are available

### Resource Verification

All resources loaded via HCS-3 MUST be verified:

- Check resource hashes
- Validate MIME types
- Enforce size limits

## Example: NFT Gallery Block

### Step 1: Store Block Definition

Store the Gutenberg block definition via HCS-1:

First, store the template HTML via HCS-1. This returns topic ID: `0.0.456788`

### Step 2: Store Block Definition

Block definition JSON:

```json
{
  "apiVersion": 3,
  "name": "hashlink/nft-gallery",
  "title": "NFT Gallery",
  "category": "hashlink/media",
  "description": "Display NFTs in a responsive gallery",
  "template_t_id": "0.0.456788",
  "attributes": {
    "collectionId": {
      "type": "string",
      "default": ""
    },
    "columns": {
      "type": "number",
      "default": 3
    },
    "showPrice": {
      "type": "boolean",
      "default": true
    }
  },
  "supports": {
    "align": ["wide", "full"],
    "spacing": {
      "margin": true,
      "padding": true
    }
  }
}
```

Store this JSON via HCS-1. This returns topic ID: `0.0.456789`

Template HTML:

```html
<div class="nft-gallery" data-columns="{{attributes.columns}}">
  {{#each nfts}}
  <div class="nft-item">
    <img src="{{this.image}}" alt="{{this.name}}" />
    <h4>{{this.name}}</h4>

    {{#if ../attributes.showPrice}}
    <p class="price">{{this.price}} ℏ</p>
    {{/if}}

    <div class="actions">
      <button data-action-click="{{actions.viewDetails}}" data-nft-id="{{this.id}}">
        View
      </button>
      <button data-action-click="{{actions.purchase}}" data-nft-id="{{this.id}}">
        Buy Now
      </button>
    </div>
  </div>
  {{/each}}
</div>
```


### Step 3: Use in Assembly

Add the block to an assembly with action bindings:

```json
{
  "p": "hcs-12",
  "op": "add-block",
  "block_t_id": "0.0.456789",
  "actions": {
    "viewDetails": "0.0.789012",
    "purchase": "0.0.789013"
  },
  "attributes": {
    "collectionId": "0.0.123456",
    "columns": 4,
    "showPrice": true
  }
}
```

## Best Practices

1. **Follow Gutenberg Standards**: Use standard Gutenberg block structure
2. **Keep Templates Simple**: Complex logic belongs in actions, not templates
3. **Use Semantic HTML**: Ensure accessibility
4. **Responsive Design**: Test on all screen sizes
5. **Version Carefully**: Blocks are immutable once registered
6. **Document Attributes**: Clear descriptions for all block settings
7. **Action Binding**:
   - Use descriptive action aliases that match their purpose
   - Validate parameters before sending to actions
   - Handle both success and error states from actions
   - Consider loading states for async action execution
   - Use data-params for static values, forms for user input
