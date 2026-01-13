# AXP - Agentic Experience Protocol

The Agentic Experience Protocol (AXP) extends UCP to enable rich product experiences in agentic commerce interfaces. AXP is based on A2A (Agent-to-Agent), ACP (Agent Communication Protocol), and UCP (Universal Commerce Protocol) standards.

**AXP uses the same security protocols and patterns as UCP**, ensuring seamless integration and consistent security posture across both protocols.

## Overview

AXP addresses three core requirements for modern agentic commerce:

1. **Product Data** - Structured product information supporting complex types (variants, configurators, events, subscriptions)
2. **Quality Data** - Trust signals including reviews, returns, intent information, and merchant reputation
3. **Experience Embedding** - Sandboxed live experiences from merchants (3D viewers, configurators, AR)

## Protocol Version

Current version: `2026-01-13`

## UCP Compatibility

AXP is designed to work as an **addon within UCP** or as a standalone protocol:

### Deployment Options

| Mode | Discovery Endpoint | Description |
|------|-------------------|-------------|
| **UCP Addon** | `/.well-known/ucp` | AXP capabilities included in UCP profile |
| **Standalone** | `/.well-known/axp` | Independent AXP discovery |
| **Both** | Both endpoints | Maximum compatibility |

### UCP Addon Example

```json
{
  "ucp": {
    "version": "2026-01-11",
    "capabilities": [
      {"name": "dev.ucp.shopping.checkout", "version": "2026-01-11", ...}
    ]
  },
  "axp": {
    "version": "2026-01-13",
    "capabilities": [
      {"name": "dev.axp.product_data", "version": "2026-01-13", ...}
    ]
  },
  "signing_keys": [...]
}
```

### UCP Item Compatibility

AXP products can be directly used in UCP checkout sessions:

```bash
# Get UCP-compatible item from AXP product
GET /api/axp/products/{id}/ucp-item

# Use in UCP checkout
POST /api/ucp/checkout-sessions
{
  "line_items": [{
    "item": { /* UCP item from AXP */ },
    "quantity": 1
  }]
}
```

## Discovery

AXP profiles are discoverable at `/.well-known/axp` (standalone) or within `/.well-known/ucp` (addon mode).

```bash
# Standalone discovery
curl https://merchant.example.com/.well-known/axp

# UCP addon discovery
curl https://merchant.example.com/.well-known/ucp
```

## Capabilities

### Product Data Capabilities

| Capability | Description |
|------------|-------------|
| `dev.axp.product_data` | Base product data access |
| `dev.axp.product_data.variants` | Variant product support |
| `dev.axp.product_data.configurator` | Complex product configurators |
| `dev.axp.product_data.events` | Event/ticket products |
| `dev.axp.product_data.subscriptions` | Subscription products |
| `dev.axp.product_data.bundles` | Product bundles |

### Quality Data Capabilities

| Capability | Description |
|------------|-------------|
| `dev.axp.quality_data` | Base quality signals |
| `dev.axp.quality_data.reviews` | Product reviews and ratings |
| `dev.axp.quality_data.returns` | Return statistics and reasons |
| `dev.axp.quality_data.intent` | Purchase intent signals |
| `dev.axp.quality_data.trust` | Merchant and product trust |

### Experience Embedding Capabilities

| Capability | Description |
|------------|-------------|
| `dev.axp.experience_embedding` | Base experience embedding |
| `dev.axp.experience_embedding.viewer_3d` | 3D product viewers |
| `dev.axp.experience_embedding.configurator` | Visual configurators |
| `dev.axp.experience_embedding.ar` | Augmented reality experiences |
| `dev.axp.experience_embedding.video` | Interactive video |
| `dev.axp.experience_embedding.custom` | Custom merchant experiences |

## Transports

AXP supports multiple transport protocols:

- **REST API** - OpenAPI 3.1 specification (`services/openapi.json`)
- **MCP** - Model Context Protocol for AI agents (`services/openrpc.json`)
- **A2A** - Agent-to-Agent protocol
- **GraphQL** - Query language support
- **WebSocket** - Real-time updates

## Schema Structure

```
product-protocol/
├── schemas/
│   ├── axp.json              # Core protocol metadata (references UCP)
│   ├── capability.json       # Capability definitions (extends UCP capabilities)
│   ├── security.json         # Security schemas (UCP-compatible)
│   ├── product.json          # Product schema
│   ├── quality_data.json     # Quality data schema
│   ├── experience.json       # Experience embedding schema
│   └── types/
│       ├── availability.json
│       ├── price.json
│       ├── variants.json
│       ├── configurator.json
│       ├── event.json
│       ├── subscription.json
│       ├── bundle.json
│       ├── digital.json
│       ├── reviews.json
│       ├── returns.json
│       ├── intent_signals.json
│       ├── merchant_trust.json
│       ├── product_trust.json
│       ├── social_proof.json
│       ├── sandbox_policy.json
│       ├── experience_entry_point.json
│       └── experience_interaction.json
├── discovery/
│   ├── profile_schema.json          # Discovery profile schema
│   └── unified_profile_example.json # Example UCP+AXP unified profile
└── services/
    ├── service_schema.json   # Service definition schema
    ├── openapi.json          # REST API specification
    └── openrpc.json          # MCP/RPC specification
```

## Product Types

AXP supports these product types:

| Type | Description |
|------|-------------|
| `simple` | Basic product with no variants |
| `variant` | Product with predefined variants (size, color) |
| `configurable` | Build-to-order with configurator |
| `bundle` | Product bundle/kit |
| `event` | Event, ticket, workshop |
| `subscription` | Recurring subscription |
| `digital` | Digital download/license |
| `service` | Service offering |

## Experience Embedding

### Sandbox Security

Embedded experiences run in secure sandboxes with configurable policies:

```json
{
  "sandbox_policy": {
    "isolation_level": "standard",
    "permissions": {
      "allow_scripts": true,
      "allow_same_origin": false,
      "allow_popups": false
    },
    "content_security_policy": {
      "default_src": ["'self'"],
      "script_src": ["'self'", "https://cdn.merchant.com"],
      "connect_src": ["'self'", "https://api.merchant.com"]
    },
    "resource_limits": {
      "max_memory_mb": 128,
      "execution_timeout_ms": 30000
    }
  }
}
```

### Entry Point Types

| Type | Description |
|------|-------------|
| `iframe` | Sandboxed iframe loading |
| `web_component` | Custom Element with Shadow DOM |
| `script` | JavaScript module loading |
| `wasm` | WebAssembly module |

### Interaction Model

Experiences communicate with the host via structured events:

**Inbound Events** (host → experience):
- `product_data_update` - Product information changed
- `variant_selected` - User selected a variant
- `locale_changed` - Language/locale changed

**Outbound Events** (experience → host):
- `ready` - Experience loaded and ready
- `configuration_changed` - User modified configuration
- `add_to_cart_request` - Request to add item to cart
- `view_changed` - 3D view angle changed
- `ar_launch_request` - Request to launch AR mode

## Quality Data

### Review Aggregates

```json
{
  "reviews": {
    "aggregate": {
      "rating": 4.3,
      "count": 1247,
      "rating_distribution": {
        "5": 782,
        "4": 312,
        "3": 98,
        "2": 34,
        "1": 21
      },
      "recommendation_percentage": 94,
      "verified_purchase_count": 1089
    },
    "aspect_ratings": [
      {"aspect": "quality", "rating": 4.5},
      {"aspect": "value", "rating": 4.1},
      {"aspect": "durability", "rating": 4.4}
    ]
  }
}
```

### Return Analysis

```json
{
  "returns": {
    "return_rate": {
      "percentage": 8.2,
      "category_comparison": "below_average",
      "trend": "improving"
    },
    "return_reasons": [
      {"reason": "wrong_size", "percentage": 45},
      {"reason": "not_as_described", "percentage": 18},
      {"reason": "changed_mind", "percentage": 22}
    ],
    "size_fit_analysis": {
      "fit_indicator": "runs_small",
      "size_recommendation": "Order one size up"
    }
  }
}
```

### Merchant Trust

```json
{
  "merchant_trust": {
    "trust_score": {
      "score": 94,
      "tier": "gold"
    },
    "verification": {
      "business_verified": true,
      "address_verified": true
    },
    "performance_metrics": {
      "on_time_delivery_rate": 97.3,
      "resolution_rate": 99.1,
      "response_time_hours": 2.4
    }
  }
}
```

## API Usage

### REST API

```bash
# Get product with quality data
curl https://merchant.example.com/api/axp/products/SKU123?include_quality=true

# Get product reviews
curl https://merchant.example.com/api/axp/products/SKU123/reviews?sort=helpful&limit=10

# Get configurator definition
curl https://merchant.example.com/api/axp/products/SKU123/configurator

# Calculate configuration price
curl -X POST https://merchant.example.com/api/axp/products/SKU123/configurator \
  -H "Content-Type: application/json" \
  -d '{"selections": {"color": "blue", "size": "medium", "engraving": "JD"}}'

# Get experience definition
curl https://merchant.example.com/api/axp/experiences/EXP456
```

### MCP (Model Context Protocol)

```json
{
  "jsonrpc": "2.0",
  "method": "axp.getProduct",
  "params": {
    "product_id": "SKU123",
    "include_quality": true,
    "include_experiences": true
  },
  "id": 1
}
```

```json
{
  "jsonrpc": "2.0",
  "method": "axp.calculateConfiguration",
  "params": {
    "product_id": "SKU123",
    "selections": {
      "color": "blue",
      "size": "medium",
      "engraving": "JD"
    }
  },
  "id": 2
}
```

## Integration with UCP

AXP extends UCP capabilities:

- AXP products can be used in UCP checkout sessions
- AXP quality data informs purchase decisions in agentic flows
- AXP experiences can be embedded in UCP-enabled platforms

```json
{
  "ucp": {
    "version": "2026-01-11",
    "capabilities": ["dev.ucp.shopping.checkout"]
  },
  "axp": {
    "version": "2026-01-13",
    "capabilities": [
      "dev.axp.product_data",
      "dev.axp.product_data.variants",
      "dev.axp.quality_data",
      "dev.axp.quality_data.reviews",
      "dev.axp.experience_embedding",
      "dev.axp.experience_embedding.viewer_3d"
    ]
  }
}
```

## Security

AXP implements the **same security protocols as UCP** to ensure consistent security and easy integration.

### Cryptographic Standards

| Component | Standard | Description |
|-----------|----------|-------------|
| **Signing Algorithm** | ES256 | ECDSA with P-256 curve and SHA-256 |
| **Key Format** | JWK | JSON Web Key (RFC 7517) |
| **Request Signatures** | Detached JWS | RFC 7797 detached signatures |
| **Tokens** | JWT | Standard JSON Web Tokens |

### Signing Keys

AXP uses the same `signing_keys` format as UCP in discovery profiles:

```json
{
  "signing_keys": [
    {
      "kid": "merchant-key-2026-01",
      "kty": "EC",
      "crv": "P-256",
      "x": "f83OJ3D2xF1Bg8vub9tLe1gHMzV76e8Tus9uPHvRVEU",
      "y": "x_FEzRu9m36HLN_tue659LNpXW6pCyStikYjKIWI5a0",
      "use": "sig",
      "alg": "ES256"
    }
  ]
}
```

When AXP is used as a UCP addon, **the same signing keys are shared** between both protocols.

### Request/Response Signatures

AXP uses the same detached JWS format as UCP:

```
Format: base64url(header)..base64url(signature)
Header: {"alg":"ES256","typ":"JWT","kid":"key-id"}
```

**HTTP Headers:**
- `Request-Signature` - Signs request body (same as UCP)
- `Response-Signature` - Signs response body
- `Content-Signature` - Signs experience content for integrity
- `AXP-Agent` - Agent identification (same format as UCP-Agent)

### Experience Content Signing

For embedded experiences, optional content signing ensures integrity:

```json
{
  "experience_signature": {
    "signature": "eyJhbGciOiJFUzI1NiJ9..signature",
    "content_hash": "sha256-base64url-hash",
    "signed_at": "2026-01-13T12:00:00Z",
    "kid": "merchant-key-2026-01"
  }
}
```

### Experience Sandboxing

- All embedded experiences run in isolated contexts
- Network access restricted to whitelisted domains
- No access to parent window or cookies
- Resource limits enforced (memory, CPU, network)
- CSP headers strictly applied

### Security Schema

See `schemas/security.json` for full security type definitions.

### Data Privacy

- Quality data aggregated to protect individual privacy
- PII handling configurable per experience
- Audit logging available for compliance

## Flow Diagrams

Detailed flow diagrams for AXP integration into UCP flows can be found in [docs/flow-diagrams.md](docs/flow-diagrams.md):

- **Scenario 1**: Product Discovery with Quality Data
- **Scenario 2**: Configurable Product with Embedded Experience
- **Scenario 3**: Checkout Flow with AXP-Enriched Products
- **Scenario 4**: Event/Ticket Booking with Time Selection
- **Scenario 5**: Subscription with Personalization
- **Scenario 6**: Trust-Based Purchase Decision
- **Scenario 7**: Complete E2E Flow

## License

Apache License 2.0
