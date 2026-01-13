# AXP Flow Diagrams

This document shows how AXP integrates into various UCP flows.

## Overview: AXP + UCP Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           AGENTIC INTERFACE                                  │
│                    (AI Assistant / Shopping Agent)                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PROTOCOL LAYER                                     │
│  ┌──────────────────────────────┐    ┌──────────────────────────────────┐   │
│  │            AXP               │    │              UCP                  │   │
│  │  • Product Data              │    │  • Checkout Sessions              │   │
│  │  • Quality Signals           │◄──►│  • Payment Processing             │   │
│  │  • Embedded Experiences      │    │  • Order Management               │   │
│  │  • Trust Information         │    │  • Identity Linking               │   │
│  └──────────────────────────────┘    └──────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MERCHANT BACKEND                                   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐    │
│  │   Product   │ │   Review    │ │ Experience  │ │   Commerce Engine   │    │
│  │   Catalog   │ │   System    │ │   Platform  │ │   (Shopware etc.)   │    │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Scenario 1: Product Discovery with Quality Data

The agent searches for products and receives quality signals for decision-making.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    User      │     │    Agent     │     │     AXP      │     │   Merchant   │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │                    │
       │ "Find running      │                    │                    │
       │  shoes"            │                    │                    │
       │───────────────────►│                    │                    │
       │                    │                    │                    │
       │                    │  axp.searchProducts│                    │
       │                    │  {query: "running  │                    │
       │                    │   shoes"}          │                    │
       │                    │───────────────────►│                    │
       │                    │                    │  GET /products     │
       │                    │                    │  ?q=running+shoes  │
       │                    │                    │───────────────────►│
       │                    │                    │                    │
       │                    │                    │◄───────────────────│
       │                    │                    │  Products[]        │
       │                    │◄───────────────────│                    │
       │                    │  Products +        │                    │
       │                    │  quality_signals   │                    │
       │                    │                    │                    │
       │                    │  axp.getQualityData│                    │
       │                    │  {product_id,      │                    │
       │                    │   include:[reviews,│                    │
       │                    │   returns, trust]} │                    │
       │                    │───────────────────►│                    │
       │                    │                    │  GET /products/    │
       │                    │                    │  {id}/quality      │
       │                    │                    │───────────────────►│
       │                    │                    │                    │
       │                    │                    │◄───────────────────│
       │                    │◄───────────────────│  Quality Data      │
       │                    │                    │                    │
       │  "Nike Air Zoom    │                    │                    │
       │   - 4.3★ (1247     │                    │                    │
       │     reviews)       │                    │                    │
       │   - 8% Return Rate │                    │                    │
       │   - Runs small"    │                    │                    │
       │◄───────────────────│                    │                    │
       │                    │                    │                    │
```

---

## Scenario 2: Configurable Product with Embedded Experience

User configures a product with a visual 3D configurator.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    User      │     │    Agent     │     │     AXP      │     │   Merchant   │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │                    │
       │ "I want to         │                    │                    │
       │  configure a       │                    │                    │
       │  shoe"             │                    │                    │
       │───────────────────►│                    │                    │
       │                    │                    │                    │
       │                    │  axp.getProduct    │                    │
       │                    │  {product_id,      │                    │
       │                    │   include_         │                    │
       │                    │   experiences:true}│                    │
       │                    │───────────────────►│                    │
       │                    │                    │───────────────────►│
       │                    │                    │◄───────────────────│
       │                    │◄───────────────────│                    │
       │                    │  Product +         │                    │
       │                    │  embedded_         │                    │
       │                    │  experiences[]     │                    │
       │                    │                    │                    │
       │                    │  axp.getExperience │                    │
       │                    │  {experience_id:   │                    │
       │                    │   "configurator-   │                    │
       │                    │    3d-shoe"}       │                    │
       │                    │───────────────────►│                    │
       │                    │                    │───────────────────►│
       │                    │                    │◄───────────────────│
       │                    │◄───────────────────│  Experience +      │
       │                    │                    │  sandbox_policy +  │
       │                    │                    │  entry_point       │
       │                    │                    │                    │
       │  ┌─────────────────────────────────────────────────────┐    │
       │  │           EMBEDDED 3D CONFIGURATOR                  │    │
       │  │  ┌───────────────────────────────────────────────┐  │    │
       │  │  │                                               │  │    │
       │  │  │     🔄 [3D Shoe Model rotating]              │  │    │
       │  │  │                                               │  │    │
       │  │  │     Color: [Blue ▼]  Size: [42 ▼]           │  │    │
       │  │  │     Material: [Mesh ▼]                       │  │    │
       │  │  │     Sole: [Running ▼]                        │  │    │
       │  │  │                                               │  │    │
       │  │  │     Price: €149.00                            │  │    │
       │  │  │     [Add to Cart]                             │  │    │
       │  │  │                                               │  │    │
       │  │  └───────────────────────────────────────────────┘  │    │
       │  │  Sandbox: iframe[sandbox="allow-scripts"]           │    │
       │  │  CSP: script-src 'self' cdn.merchant.com           │    │
       │  └─────────────────────────────────────────────────────┘    │
       │                    │                    │                    │
       │  User interacts    │                    │                    │
       │  with configurator │                    │                    │
       │──────────────────► │                    │                    │
       │                    │                    │                    │
       │                    │  Experience emits: │                    │
       │                    │  "configuration_   │                    │
       │                    │   changed"         │                    │
       │                    │◄─ ─ ─ ─ ─ ─ ─ ─ ─ │                    │
       │                    │                    │                    │
       │                    │  axp.calculate     │                    │
       │                    │  Configuration     │                    │
       │                    │  {selections}      │                    │
       │                    │───────────────────►│                    │
       │                    │                    │───────────────────►│
       │                    │                    │◄───────────────────│
       │                    │◄───────────────────│  {price, lead_time,│
       │                    │                    │   availability}    │
       │                    │                    │                    │
```

---

## Scenario 3: Checkout Flow with AXP-Enriched Products

Complete flow from product selection to checkout.

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                                DISCOVERY PHASE                                      │
│                                    (AXP)                                           │
└────────────────────────────────────────────────────────────────────────────────────┘
                                        │
       ┌────────────────────────────────┼────────────────────────────────┐
       │                                │                                │
       ▼                                ▼                                ▼
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│  Product Search │          │  Quality Check  │          │   Experience    │
│                 │          │                 │          │                 │
│ axp.search      │          │ axp.getQuality  │          │ axp.get         │
│ Products()      │          │ Data()          │          │ Experience()    │
│                 │          │ • Reviews       │          │ • 3D Viewer     │
│ Returns:        │          │ • Returns       │          │ • AR Try-on     │
│ • Products[]    │          │ • Trust         │          │ • Configurator  │
│ • quality_      │          │ • Intent        │          │                 │
│   signals       │          │                 │          │                 │
└────────┬────────┘          └────────┬────────┘          └────────┬────────┘
         │                            │                            │
         └────────────────────────────┼────────────────────────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  User Decision      │
                           │  (informed by       │
                           │   quality data)     │
                           │                     │
                           │  "Buy"              │
                           └──────────┬──────────┘
                                      │
                                      ▼
┌────────────────────────────────────────────────────────────────────────────────────┐
│                                CHECKOUT PHASE                                       │
│                                    (UCP)                                           │
└────────────────────────────────────────────────────────────────────────────────────┘
                                      │
       ┌──────────────────────────────┼──────────────────────────────┐
       │                              │                              │
       ▼                              ▼                              ▼
┌─────────────────┐          ┌─────────────────┐          ┌─────────────────┐
│  Create Session │          │  Add Payment    │          │   Complete      │
│                 │          │                 │          │                 │
│ POST /checkout- │          │ PUT /checkout-  │          │ POST /checkout- │
│ sessions        │          │ sessions/{id}   │          │ sessions/{id}/  │
│                 │          │                 │          │ complete        │
│ {               │          │ {payment:       │          │                 │
│  line_items: [  │          │  {handler,      │          │ Returns:        │
│   {item: {      │─────────►│   credential}}  │─────────►│ • order_id      │
│     id: "AXP-   │          │                 │          │ • confirmation  │
│      PRODUCT",  │          │ Returns:        │          │                 │
│     title,      │          │ • status:       │          │                 │
│     price       │          │   "ready_for_   │          │                 │
│   }}           │          │    complete"    │          │                 │
│  ],             │          │                 │          │                 │
│  buyer: {...}   │          │                 │          │                 │
│ }               │          │                 │          │                 │
└─────────────────┘          └─────────────────┘          └─────────────────┘
```

---

## Scenario 4: Event/Ticket Booking with Time Selection

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    User      │     │    Agent     │     │   AXP + UCP  │     │   Merchant   │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │                    │
       │ "Book tickets for  │                    │                    │
       │  the concert"      │                    │                    │
       │───────────────────►│                    │                    │
       │                    │                    │                    │
       │                    │ ┌─────────────────────────────────────┐ │
       │                    │ │              AXP                    │ │
       │                    │ └─────────────────────────────────────┘ │
       │                    │                    │                    │
       │                    │  axp.getProduct    │                    │
       │                    │  (type: "event")   │                    │
       │                    │───────────────────►│                    │
       │                    │                    │───────────────────►│
       │                    │                    │                    │
       │                    │◄───────────────────│◄───────────────────│
       │                    │  Event Product:    │                    │
       │                    │  • occurrences[]   │                    │
       │                    │  • ticket_types[]  │                    │
       │                    │  • venue           │                    │
       │                    │  • capacity        │                    │
       │                    │                    │                    │
       │  "Available        │                    │                    │
       │   dates:           │                    │                    │
       │   - Fri 01/20 8pm  │                    │                    │
       │     (142 seats)    │                    │                    │
       │   - Sat 01/21 8pm  │                    │                    │
       │     (89 seats)"    │                    │                    │
       │◄───────────────────│                    │                    │
       │                    │                    │                    │
       │ "Saturday, 2 VIP   │                    │                    │
       │  tickets"          │                    │                    │
       │───────────────────►│                    │                    │
       │                    │                    │                    │
       │                    │ ┌─────────────────────────────────────┐ │
       │                    │ │              UCP                    │ │
       │                    │ └─────────────────────────────────────┘ │
       │                    │                    │                    │
       │                    │  POST /checkout-   │                    │
       │                    │  sessions          │                    │
       │                    │  {line_items: [{   │                    │
       │                    │    item: {         │                    │
       │                    │     id: "event-    │                    │
       │                    │      occurrence-   │                    │
       │                    │      sat-21",      │                    │
       │                    │     variant_id:    │                    │
       │                    │      "vip-ticket"  │                    │
       │                    │    },              │                    │
       │                    │    quantity: 2     │                    │
       │                    │  }]}               │                    │
       │                    │───────────────────►│                    │
       │                    │                    │───────────────────►│
       │                    │                    │                    │
       │                    │◄───────────────────│◄───────────────────│
       │                    │  Checkout Session  │                    │
       │                    │                    │                    │
       │  "Checkout ready:  │                    │                    │
       │   2x VIP Ticket    │                    │                    │
       │   Sat 01/21, €298" │                    │                    │
       │◄───────────────────│                    │                    │
```

---

## Scenario 5: Subscription with Personalization

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                           SUBSCRIPTION FLOW                                   │
└──────────────────────────────────────────────────────────────────────────────┘

    ┌───────────────┐        ┌───────────────┐        ┌───────────────┐
    │   DISCOVER    │        │   CUSTOMIZE   │        │   CHECKOUT    │
    │     (AXP)     │───────►│     (AXP)     │───────►│     (UCP)     │
    └───────────────┘        └───────────────┘        └───────────────┘
           │                        │                        │
           ▼                        ▼                        ▼
    ┌─────────────────────────────────────────────────────────────────────┐
    │                                                                     │
    │  axp.getProduct              axp.getExperience      POST /checkout │
    │  (type:"subscription")       ("subscription-        -sessions      │
    │                               customizer")                         │
    │  Returns:                                           {              │
    │  • pricing_plans[]           ┌─────────────────┐     line_items:[  │
    │  • delivery_frequency        │ Embedded        │     {subscription:│
    │  • pause_options             │ Customizer      │      {plan_id,    │
    │  • customization             │                 │       frequency,  │
    │                              │ ☐ Protein Bar  │       selections}}│
    │  Intent Signals:             │ ☐ Energy Gel   │     ]             │
    │  • 73% choose monthly        │ ☑ Recovery Mix │    }              │
    │  • Popular: "Athlete         │ ☐ Hydration    │                   │
    │    Bundle"                   │                 │    Returns:       │
    │                              │ Frequency:      │    • subscription_│
    │  Quality Data:               │ [Monthly ▼]    │      id           │
    │  • 4.6★ average              │                 │    • next_billing │
    │  • 2.1% churn rate           │ [Continue]      │    • first_       │
    │                              └─────────────────┘      delivery     │
    │                                                                     │
    └─────────────────────────────────────────────────────────────────────┘
```

---

## Scenario 6: Trust-Based Purchase Decision

The agent uses quality data to support the user in making a purchase decision.

```
                              ┌─────────────────────┐
                              │    User Intent:     │
                              │ "Should I buy from  │
                              │  this merchant?"    │
                              │                     │
                              └──────────┬──────────┘
                                         │
                                         ▼
              ┌──────────────────────────────────────────────┐
              │                    AGENT                      │
              │         Collects Trust Information            │
              └──────────────────────────────────────────────┘
                                         │
           ┌─────────────────────────────┼─────────────────────────────┐
           │                             │                             │
           ▼                             ▼                             ▼
┌─────────────────────┐      ┌─────────────────────┐      ┌─────────────────────┐
│  MERCHANT TRUST     │      │   PRODUCT TRUST     │      │   SOCIAL PROOF      │
│       (AXP)         │      │       (AXP)         │      │       (AXP)         │
├─────────────────────┤      ├─────────────────────┤      ├─────────────────────┤
│                     │      │                     │      │                     │
│ axp.getMerchant     │      │ axp.getQualityData  │      │ axp.getQualityData  │
│ Trust()             │      │ (product_trust)     │      │ (social_proof)      │
│                     │      │                     │      │                     │
│ ┌─────────────────┐ │      │ ┌─────────────────┐ │      │ ┌─────────────────┐ │
│ │ Trust Score: 94 │ │      │ │ Authenticity:   │ │      │ │ 1,247 Reviews   │ │
│ │ Tier: Gold      │ │      │ │ ✓ Verified      │ │      │ │ 94% recommend   │ │
│ │                 │ │      │ │                 │ │      │ │                 │ │
│ │ ✓ Verified Biz  │ │      │ │ ✓ Authorized    │ │      │ │ Featured in:    │ │
│ │ ✓ 5 years in    │ │      │ │   Retailer      │ │      │ │ • TechCrunch    │ │
│ │   business      │ │      │ │                 │ │      │ │ • Wired         │ │
│ │                 │ │      │ │ Safety:         │ │      │ │                 │ │
│ │ Performance:    │ │      │ │ • CE certified  │ │      │ │ Awards:         │ │
│ │ • 97% on-time   │ │      │ │ • No recalls    │ │      │ │ • Best Product  │ │
│ │ • 2.4h response │ │      │ │                 │ │      │ │   2025          │ │
│ │ • 99% resolved  │ │      │ │                 │ │      │ │                 │ │
│ └─────────────────┘ │      │ └─────────────────┘ │      │ └─────────────────┘ │
│                     │      │                     │      │                     │
└─────────────────────┘      └─────────────────────┘      └─────────────────────┘
           │                             │                             │
           └─────────────────────────────┼─────────────────────────────┘
                                         │
                                         ▼
              ┌──────────────────────────────────────────────┐
              │                 AGENT RESPONSE               │
              │                                              │
              │  "Yes, this merchant is trustworthy:         │
              │                                              │
              │   ✓ Gold-Tier Merchant (94/100 Trust Score)  │
              │   ✓ Verified business for 5 years            │
              │   ✓ 97% on-time delivery                     │
              │   ✓ Authorized retailer for this brand       │
              │   ✓ Product CE-certified, no recalls         │
              │   ✓ 1,247 reviews, 94% recommend             │
              │                                              │
              │   Would you like to proceed?"                │
              └──────────────────────────────────────────────┘
```

---

## Scenario 7: Complete E2E Flow

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                              END-TO-END AGENTIC COMMERCE                            │
└────────────────────────────────────────────────────────────────────────────────────┘

 USER                    AGENT                     AXP                      UCP
  │                        │                        │                        │
  │  "I need new           │                        │                        │
  │   headphones for       │                        │                        │
  │   sports, max €150"    │                        │                        │
  │───────────────────────►│                        │                        │
  │                        │                        │                        │
  │                        │ ┌──────────────────────────────────────────┐    │
  │                        │ │ 1. DISCOVERY                             │    │
  │                        │ └──────────────────────────────────────────┘    │
  │                        │                        │                        │
  │                        │  axp.searchProducts   │                        │
  │                        │───────────────────────►│                        │
  │                        │                        │                        │
  │                        │◄───────────────────────│  Products + Quality    │
  │                        │                        │                        │
  │                        │ ┌──────────────────────────────────────────┐    │
  │                        │ │ 2. QUALITY ASSESSMENT                    │    │
  │                        │ └──────────────────────────────────────────┘    │
  │                        │                        │                        │
  │                        │  axp.getQualityData   │                        │
  │                        │  (for top 3 products) │                        │
  │                        │───────────────────────►│                        │
  │                        │                        │                        │
  │                        │◄───────────────────────│  Reviews + Returns     │
  │                        │                        │  + Trust Data          │
  │                        │                        │                        │
  │  "Top 3 recommendations:                        │                        │
  │   1. Sony WF-SP800N    │                        │                        │
  │      €129, 4.5★        │                        │                        │
  │      5% Return Rate    │                        │                        │
  │   ..."                 │                        │                        │
  │◄───────────────────────│                        │                        │
  │                        │                        │                        │
  │  "Show me the Sony     │                        │                        │
  │   in detail"           │                        │                        │
  │───────────────────────►│                        │                        │
  │                        │                        │                        │
  │                        │ ┌──────────────────────────────────────────┐    │
  │                        │ │ 3. EXPERIENCE                            │    │
  │                        │ └──────────────────────────────────────────┘    │
  │                        │                        │                        │
  │                        │  axp.getExperience    │                        │
  │                        │  ("3d-viewer-sony")   │                        │
  │                        │───────────────────────►│                        │
  │                        │                        │                        │
  │                        │◄───────────────────────│  Experience +          │
  │                        │                        │  Sandbox Policy        │
  │                        │                        │                        │
  │  ┌─────────────────────────────────────────┐   │                        │
  │  │  [3D VIEWER - SANDBOXED]                │   │                        │
  │  │  🎧 Sony WF-SP800N                      │   │                        │
  │  │     [Rotate] [Zoom] [AR View]           │   │                        │
  │  └─────────────────────────────────────────┘   │                        │
  │◄───────────────────────│                        │                        │
  │                        │                        │                        │
  │  "I'll take it,        │                        │                        │
  │   in black"            │                        │                        │
  │───────────────────────►│                        │                        │
  │                        │                        │                        │
  │                        │ ┌──────────────────────────────────────────────┐
  │                        │ │ 4. CHECKOUT                                  │
  │                        │ └──────────────────────────────────────────────┘
  │                        │                        │                        │
  │                        │                        │  POST /checkout-       │
  │                        │                        │  sessions              │
  │                        │────────────────────────┼───────────────────────►│
  │                        │                        │                        │
  │                        │◄───────────────────────┼────────────────────────│
  │                        │                        │  Session Created       │
  │                        │                        │                        │
  │                        │                        │  PUT /checkout-        │
  │                        │                        │  sessions/{id}         │
  │                        │                        │  (add payment)         │
  │                        │────────────────────────┼───────────────────────►│
  │                        │                        │                        │
  │                        │◄───────────────────────┼────────────────────────│
  │                        │                        │  ready_for_complete    │
  │                        │                        │                        │
  │  "Checkout ready:      │                        │                        │
  │   Sony WF-SP800N       │                        │                        │
  │   Black, €129          │                        │                        │
  │   Delivery: 2-3 days   │                        │                        │
  │   [Confirm]"           │                        │                        │
  │◄───────────────────────│                        │                        │
  │                        │                        │                        │
  │  [Confirm]             │                        │                        │
  │───────────────────────►│                        │                        │
  │                        │                        │                        │
  │                        │                        │  POST /checkout-       │
  │                        │                        │  sessions/{id}/        │
  │                        │                        │  complete              │
  │                        │────────────────────────┼───────────────────────►│
  │                        │                        │                        │
  │                        │◄───────────────────────┼────────────────────────│
  │                        │                        │  Order Confirmation    │
  │                        │                        │                        │
  │  "✅ Order placed!      │                        │                        │
  │   Order #12345"        │                        │                        │
  │◄───────────────────────│                        │                        │
  │                        │                        │                        │
```

---

## Protocol Integration Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AXP + UCP INTEGRATION MATRIX                          │
├───────────────────────┬───────────────────────┬─────────────────────────────┤
│       PHASE           │      PROTOCOL         │         CAPABILITIES        │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│                       │                       │                             │
│  Product Discovery    │         AXP           │  • searchProducts           │
│                       │                       │  • getProduct               │
│                       │                       │  • getVariants              │
│                       │                       │                             │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│                       │                       │                             │
│  Quality Assessment   │         AXP           │  • getQualityData           │
│                       │                       │  • getReviews               │
│                       │                       │  • getMerchantTrust         │
│                       │                       │                             │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│                       │                       │                             │
│  Configuration        │         AXP           │  • getConfigurator          │
│                       │                       │  • calculateConfiguration   │
│                       │                       │                             │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│                       │                       │                             │
│  Experience           │         AXP           │  • getExperience            │
│  Embedding            │                       │  • listExperiences          │
│                       │                       │  • sandbox_policy           │
│                       │                       │                             │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│                       │                       │                             │
│  Cart & Checkout      │         UCP           │  • createCheckoutSession    │
│                       │                       │  • updateCheckoutSession    │
│                       │                       │  • completeCheckout         │
│                       │                       │                             │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│                       │                       │                             │
│  Payment              │         UCP           │  • Payment Handlers         │
│                       │                       │  • Tokenization             │
│                       │                       │  • AP2 Mandates             │
│                       │                       │                             │
├───────────────────────┼───────────────────────┼─────────────────────────────┤
│                       │                       │                             │
│  Order Management     │         UCP           │  • Order Webhooks           │
│                       │                       │  • Fulfillment Updates      │
│                       │                       │                             │
└───────────────────────┴───────────────────────┴─────────────────────────────┘
```
