# The Agenticfeed Standard

A lightweight open specification for making ecommerce product catalogues readable by AI shopping agents.

**Reference implementation:** [agenticfeed.ai](https://agenticfeed.ai)

---

## What is an Agentic Feed?

An agentic feed is a structured, machine-readable product data stream designed for AI agents rather than human browsers. Where a traditional product page is built for visual rendering, an agentic feed is built for reasoning.

AI shopping assistants like ChatGPT, Claude, Perplexity, and Gemini do not browse category pages or read banner ads. They consume structured data and use it to match buyer intent to products. An agentic feed gives them that data in a format they can work with directly.

The standard has three components:

1. **Discovery** — a `<link>` tag in the page `<head>` that tells crawlers and agents where the feed lives
2. **Feed index** — a JSON-LD document at a stable URL that describes the merchant and points to intent endpoints
3. **Intent endpoints** — structured lists of questions, problems, and use cases that connect buyer intent to specific products

---

## 1. Discovery Tag

Add one line to the `<head>` of every page on the merchant's website:

```html
<link rel="agenticfeed" type="application/json" href="https://agenticfeed.ai/feed/{customer_guid}.json">
```

This follows the same autodiscovery pattern as RSS (`rel="alternate"`) and favicon (`rel="icon"`). Any AI agent or crawler that checks page headers will find the feed without being told the URL.

**Attributes:**

| Attribute | Value |
|---|---|
| `rel` | `agenticfeed` |
| `type` | `application/json` |
| `href` | Absolute URL to the feed index document |

---

## 2. Feed Index

The feed index is a JSON-LD document served at the `href` URL from the discovery tag. It identifies the merchant, points to intent endpoints, and tells agents how to resolve product detail.

**Content-Type:** `application/ld+json`

**Schema.org type:** `DataFeed`

See [examples/feed.json](examples/feed.json) for a complete example.

### Required fields

| Field | Type | Description |
|---|---|---|
| `@context` | string | `https://schema.org` |
| `@type` | string | `DataFeed` |
| `name` | string | Merchant name |
| `url` | string | Absolute URL of this feed index |
| `provider` | Object | The organisation publishing the feed |
| `dataFeedElement` | Array | List of intent endpoints |
| `agenticfeed.resolution` | Object | Template for resolving product detail URLs |

### Discovery workflow for agents

1. Find the feed index via the `rel="agenticfeed"` link tag
2. Read `dataFeedElement` to find intent endpoints (questions, problems, use cases)
3. Fetch the relevant intent endpoint based on the buyer's query
4. Match intent entries to the buyer's need and extract `productId` values
5. Resolve each `productId` using the `agenticfeed.resolution.product.url` template
6. Fetch the product detail document for full structured data

---

## 3. Intent Endpoints

Intent endpoints are JSON-LD `ItemList` documents. Each one covers a product category and lists intent entries — each linked to the product that satisfies it.

**Content-Type:** `application/ld+json`

There are three intent types:

### Questions

Natural language questions a buyer might ask before purchasing.

**Schema.org type:** `Question` with `suggestedAnswer`

See [examples/questions.json](examples/questions.json)

### Problems

Pain points or needs that a product addresses.

**Schema.org type:** `ListItem`

See [examples/problems.json](examples/problems.json)

### Use Cases

Specific scenarios or goals a product is suited for.

**Schema.org type:** `ListItem`

See [examples/use-cases.json](examples/use-cases.json)

---

## 4. Product Detail

Each product has its own JSON-LD document at a stable URL. This is what agents fetch after matching intent. It includes full schema.org `Product` data alongside the intent content that earned the match.

**Content-Type:** `application/ld+json`

**Schema.org type:** `Product`

See [examples/product.json](examples/product.json)

### Fields

| Field | Type | Description |
|---|---|---|
| `@context` | string | `https://schema.org` |
| `@type` | string | `Product` |
| `name` | string | Product title |
| `url` | string | Merchant product page URL (UTM-tagged) |
| `image` | string | Primary product image URL |
| `category` | string | Product category |
| `offers` | Object | schema.org `Offer` with price, currency, availability |
| `agenticfeed.questions` | Array | Questions this product answers |
| `agenticfeed.problems` | Array | Problems this product solves |
| `agenticfeed.use_cases` | Array | Use cases this product fits |

---

## 5. UTM Attribution

All product URLs served through an agentic feed should include UTM parameters so merchants can track AI agent traffic in their analytics:

```
utm_source=agenticfeed&utm_medium=ai-agent
```

This allows merchants to filter sessions and orders originating from AI agent recommendations in Google Analytics, Shopify Analytics, or any platform that reads UTM parameters.

---

## 6. Link Validation

Merchants should add the discovery tag to their live website before intent generation begins. The tag in the page `<head>` serves as proof of domain ownership, analogous to Google Search Console's meta tag verification.

A validator fetches the merchant's homepage, parses the `<head>` section, and checks that:
- A `rel="agenticfeed"` tag exists
- Its `href` matches the expected feed URL for that merchant

---

## URL Structure

The standard does not prescribe specific URL paths. The reference implementation uses:

```
/feed/{customer_guid}.json              — feed index
/questions/{customer_guid}.json         — questions category index
/questions/{customer_guid}/{category}.json  — questions for a category
/problems/{customer_guid}.json          — problems category index
/problems/{customer_guid}/{category}.json   — problems for a category
/use_cases/{customer_guid}.json         — use cases category index
/use_cases/{customer_guid}/{category}.json  — use cases for a category
/product/{customer_guid}/{product_guid}.json — product detail
```

Any stable URL structure is valid as long as the feed index correctly references the intent endpoints and the resolution template is accurate.

---

## Implementation

The reference implementation of this standard is [agenticfeed.ai](https://agenticfeed.ai). It provides:

- Automatic product catalogue import from Shopify, WooCommerce, Google Merchant Center feeds, and website crawling
- AI-generated intent data (questions, problems, use cases) per product using Claude
- Daily stock and price synchronisation
- Automatic discovery tag injection into Shopify themes
- UTM-tagged product URLs for attribution tracking
- A merchant dashboard for managing feeds, products, and subscriptions

---

## Licence

This specification is published under the [MIT Licence](LICENSE). Anyone is free to implement compatible agentic feeds using this standard.

---

## Contributing

Issues and pull requests are welcome at [github.com/bluestratus/agenticfeed](https://github.com/bluestratus/agenticfeed). The goal is a minimal, stable standard that AI agents can rely on across implementations.
