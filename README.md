# Agenticfeed Standard

[![Version](https://img.shields.io/badge/version-0.1.0-72e0a8)](https://github.com/bluestratus/agenticfeed/releases/tag/v0.1.0)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Reference implementation](https://img.shields.io/badge/reference_implementation-agenticfeed.ai-0d0d1a)](https://agenticfeed.ai)

**An open specification for making ecommerce product catalogues readable by AI shopping agents.**

AI assistants like ChatGPT, Claude, Perplexity, and Gemini are already recommending products to millions of shoppers. They do not browse category pages or read banner ads. They consume structured data and reason about which products best match the buyer's intent. Most merchant websites are invisible to them.

This standard defines a lightweight, discoverable format that gives AI agents exactly what they need.

---

## Quick start

Add one line to the `<head>` of every page on your website:

```html
<link rel="agenticfeed" type="application/json" href="https://yourdomain.com/feed.json">
```

That tag tells any AI agent or crawler where your structured product feed lives. The rest of this document describes what that feed should contain.

---

## Contents

- [What is an agentic feed?](#what-is-an-agentic-feed)
- [Why does this standard exist?](#why-does-this-standard-exist)
- [How is it different from a Google Merchant Center feed?](#how-is-it-different-from-a-google-merchant-center-feed)
- [How do AI agents discover it?](#how-do-ai-agents-discover-it)
- [How do I add it to my website?](#how-do-i-add-it-to-my-website)
- [Specification](#specification)
  - [1. Discovery tag](#1-discovery-tag)
  - [2. Feed index](#2-feed-index)
  - [3. Intent endpoints](#3-intent-endpoints)
  - [4. Product detail](#4-product-detail)
  - [5. UTM attribution](#5-utm-attribution)
  - [6. Link validation](#6-link-validation)
- [Agent Query API](#agent-query-api)
  - [Authentication](#authentication)
  - [Request](#request)
  - [Response](#response)
  - [Agent Card](#agent-card)
- [Examples](#examples)
- [Reference implementation](#reference-implementation)
- [Contributing](#contributing)
- [Licence](#licence)

---

## What is an agentic feed?

An agentic feed is a structured product data feed built for AI agents rather than search engine crawlers or human browsers.

A traditional product page is designed to be rendered and read by a person. A Google Merchant Center feed is designed to be parsed by a price comparison engine. An agentic feed is designed to be reasoned about by an AI.

The key difference is intent data. Where a merchant feed tells an agent "here is a cordless drill, it costs £29.99 and it is in stock," an agentic feed tells it "here is a cordless drill that answers the question *what drill do I need for assembling flat-pack furniture*, solves the problem *I keep stripping screws with my old drill*, and fits the use case *home DIY for a first-time homeowner*."

That is the layer of context an AI agent needs to make a confident recommendation to a specific buyer.

---

## Why does this standard exist?

AI shopping is already happening. The tooling to serve it well does not yet exist as an open, portable standard.

Search engines standardised web content discovery through sitemaps, robots.txt, and canonical tags. RSS standardised content syndication through a single autodiscovery tag. Neither was designed for the kind of structured reasoning that AI agents perform when they decide what to recommend.

This specification fills that gap. It defines:

- How a website signals to AI agents that a structured product feed exists
- What format that feed takes
- How intent data (questions, problems, use cases) is structured alongside standard product data
- How AI agents navigate from a buyer's query to a specific product recommendation

The format is intentionally minimal. It builds on schema.org types that crawlers already understand. It adds the intent layer that makes AI recommendation possible.

---

## How is it different from a Google Merchant Center feed?

| | Google Merchant Center feed | Agenticfeed |
|---|---|---|
| Format | XML (RSS-like) | JSON-LD |
| Schema | Google's proprietary spec | schema.org + agenticfeed namespace |
| Primary consumer | Price comparison, Shopping ads | AI agents, LLMs, shopping assistants |
| Discovery | Manual URL submission | Autodiscovery via `<link rel="agenticfeed">` |
| Intent data | None | Questions, problems, use cases per product |
| Product context | Price, title, image, availability | Price, title, image + buyer intent layer |
| Attribution | None built in | UTM parameters on every product URL |

A Google Merchant Center feed tells a machine *what* a product is. An agentic feed tells it *why a specific buyer should choose it*.

The two are complementary. Many merchants use a GMC feed as the data source for generating an agentic feed.

---

## How do AI agents discover it?

The discovery mechanism follows the same autodiscovery pattern the web has used for decades:

```
rel="stylesheet"   tells browsers where to find CSS
rel="icon"         tells browsers where to find the favicon
rel="alternate"    tells crawlers where to find RSS feeds
rel="agenticfeed"  tells AI agents where to find structured product data
```

When an AI agent or crawler visits a merchant's website, it reads the page `<head>`. If it finds a `rel="agenticfeed"` tag, it knows exactly where to fetch structured product data without being told the URL in advance.

This means:

1. No manual registration with each AI platform
2. No API keys or access agreements required
3. Any AI agent that implements this standard can discover your feed automatically
4. The merchant controls the data at their own URL

---

## How do I add it to my website?

**Step 1.** Add the discovery tag to every page `<head>`:

```html
<link rel="agenticfeed" type="application/json" href="https://yourdomain.com/feed.json">
```

**Step 2.** Serve a JSON-LD feed index at that URL (see [examples/feed.json](examples/feed.json)):

```json
{
  "@context": "https://schema.org",
  "@type": "DataFeed",
  "name": "Your Store Name",
  "url": "https://yourdomain.com/feed.json",
  "provider": {
    "@type": "Organization",
    "name": "Agenticfeed",
    "url": "https://agenticfeed.ai"
  },
  "dataFeedElement": [
    {
      "@type": "DataFeedItem",
      "name": "Questions",
      "url": "https://yourdomain.com/questions.json"
    },
    {
      "@type": "DataFeedItem",
      "name": "Problems",
      "url": "https://yourdomain.com/problems.json"
    },
    {
      "@type": "DataFeedItem",
      "name": "Use Cases",
      "url": "https://yourdomain.com/use-cases.json"
    }
  ]
}
```

**Step 3.** Serve intent endpoints for each category (see [examples/questions.json](examples/questions.json)).

**Step 4.** Serve product detail documents for each product (see [examples/product.json](examples/product.json)).

For Shopify merchants, [agenticfeed.ai](https://agenticfeed.ai) handles all of this automatically including injecting the discovery tag into your theme.

---

## Specification

### 1. Discovery tag

```html
<link rel="agenticfeed" type="application/json" href="{absolute-url-to-feed-index}">
```

| Attribute | Value |
|---|---|
| `rel` | `agenticfeed` |
| `type` | `application/json` |
| `href` | Absolute URL to the feed index document |

Place this tag in the `<head>` of every page. It must appear in the server-rendered HTML, not injected by JavaScript, so crawlers can find it without executing scripts.

---

### 2. Feed index

**Content-Type:** `application/ld+json`
**Schema.org type:** `DataFeed`

The feed index is the entry point for any agent reading your feed. It identifies the merchant, lists intent endpoints, and provides the URL template for resolving individual products.

See [examples/feed.json](examples/feed.json) for a complete example.

**Agent workflow:**

1. Find the discovery tag in the page `<head>`
2. Fetch the feed index
3. Read `dataFeedElement` to find intent endpoints
4. Fetch the relevant endpoint based on the buyer's query type
5. Match intent entries to the buyer's need and extract product IDs
6. Resolve each product ID using the `agenticfeed.resolution.product.url` template
7. Fetch the product detail document

---

### 3. Intent endpoints

Intent endpoints are JSON-LD `ItemList` documents grouped by product category. There are three intent types:

**Questions** — natural language questions a buyer asks before purchasing.
Schema.org type: `Question` with `suggestedAnswer` pointing to the product URL.
See [examples/questions.json](examples/questions.json)

**Problems** — pain points or needs the product addresses.
Schema.org type: `ListItem` with `name` (the problem) and `url` (the product).
See [examples/problems.json](examples/problems.json)

**Use cases** — specific scenarios or goals the product fits.
Schema.org type: `ListItem` with `name` (the use case) and `url` (the product).
See [examples/use-cases.json](examples/use-cases.json)

---

### 4. Product detail

**Content-Type:** `application/ld+json`
**Schema.org type:** `Product`

Each product has its own document at a stable URL. It combines standard schema.org `Product` data with the intent content under an `agenticfeed` namespace.

See [examples/product.json](examples/product.json) for a complete example.

| Field | Type | Description |
|---|---|---|
| `@context` | string | `https://schema.org` |
| `@type` | string | `Product` |
| `name` | string | Product title |
| `url` | string | Merchant product page URL (UTM-tagged) |
| `image` | string | Primary product image URL |
| `category` | string | Product category |
| `offers` | Object | schema.org `Offer` — price, currency, availability |
| `agenticfeed.questions` | Array | Questions this product answers |
| `agenticfeed.problems` | Array | Problems this product solves |
| `agenticfeed.use_cases` | Array | Use cases this product fits |

---

### 5. UTM attribution

All product URLs in an agentic feed should carry UTM parameters so merchants can measure AI agent traffic in their analytics:

```
utm_source=agenticfeed&utm_medium=ai-agent
```

This lets merchants filter and report on sessions and orders that originated from an AI agent recommendation in Google Analytics, Shopify Analytics, or any platform that reads UTM parameters.

---

### 6. Link validation

The discovery tag in the merchant's `<head>` serves as proof of domain ownership before a feed goes live. A validator fetches the merchant's homepage, parses the `<head>`, and confirms:

- A `rel="agenticfeed"` tag is present
- Its `href` matches the expected feed URL for that merchant

This prevents one merchant from claiming another merchant's domain in a feed.

---

## Agent Query API

The passive feed endpoints let AI agents crawl product data at their own pace. The Agent Query API is the active layer — an AI agent or application can send a natural language buyer intent and receive back a ranked list of matched products with reasons.

This is agent-to-agent communication. Instead of a human typing into a search box, one AI asks another AI to find the best matching products.

**Endpoint**

```
POST https://agenticfeed.ai/agent/query
Content-Type: application/json
Authorization: Bearer af_your_api_key
```

---

### Authentication

Two keys are required:

**1. Agenticfeed API key** — passed as a `Bearer` token in the `Authorization` header. This identifies who is making the request and controls access to the feed data. Generate one at [agenticfeed.ai/dashboard.html](https://agenticfeed.ai/dashboard.html).

**2. Anthropic API key** — passed in the request body as `anthropic_api_key`. This is your own key from [console.anthropic.com](https://console.anthropic.com). Agenticfeed uses it to run the matching call against Claude. You pay Anthropic directly for the AI compute — Agenticfeed does not charge for or subsidise this.

If you omit `anthropic_api_key`, the request will be rejected unless the implementation has a fallback key configured. The reference implementation requires callers to supply their own.

Your Anthropic key is validated with a minimal one-token call before any database work is done. An invalid key is rejected immediately with a clear error.

---

### Request

```json
{
  "query": "ergonomic chair for someone with lower back pain, works from home 10 hours a day, small home office, budget under £300",
  "anthropic_api_key": "sk-ant-your-key",
  "customer_guid": "a9a8378c94",
  "limit": 5
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Natural language description of what the buyer needs |
| `anthropic_api_key` | string | Yes | Your Anthropic API key — you pay for the AI call |
| `customer_guid` | string | No | The merchant feed to query. Defaults to the feed linked to your API key |
| `limit` | integer | No | Maximum results to return (1–10, default 5) |

---

### Response

Returns a schema.org `ItemList` of matched `Product` entries, each with a `match_reason` explaining why that product fits the buyer's need.

**Content-Type:** `application/ld+json`

```json
{
  "@context": "https://schema.org",
  "@type": "ItemList",
  "query": "ergonomic chair for back pain...",
  "numberOfItems": 3,
  "itemListElement": [
    {
      "@type": "Product",
      "position": 1,
      "name": "HM Seating Contessa Chair",
      "url": "https://merchant.com/contessa?utm_source=agenticfeed&utm_medium=ai-agent",
      "image": "https://merchant.com/images/contessa.jpg",
      "category": "office chairs",
      "offers": {
        "@type": "Offer",
        "price": "249.00",
        "priceCurrency": "GBP",
        "availability": "https://schema.org/InStock"
      },
      "agenticfeed": {
        "product_id": "b3f1e29a4c",
        "match_reason": "Matches adjustable lumbar support, compact footprint for small spaces, and falls within the stated budget."
      }
    }
  ]
}
```

The `match_reason` field is designed to be passed directly to the end buyer by the calling agent. It explains the recommendation in plain language without requiring the agent to do any additional reasoning.

---

### Agent Card

The reference implementation publishes an A2A-compatible Agent Card at:

```
GET https://agenticfeed.ai/.well-known/agent.json
```

This file describes the agent's capabilities, accepted inputs, output format, and authentication requirements in a machine-readable format. AI platforms that implement Google's Agent-to-Agent (A2A) protocol can discover and call the query endpoint automatically using this card.

---

## Examples

All examples are in the [examples/](examples/) directory:

| File | Description |
|---|---|
| [examples/feed.json](examples/feed.json) | Complete feed index |
| [examples/product.json](examples/product.json) | Product detail with intent data |
| [examples/questions.json](examples/questions.json) | Questions endpoint |
| [examples/problems.json](examples/problems.json) | Problems endpoint |
| [examples/use-cases.json](examples/use-cases.json) | Use cases endpoint |
| [examples/link-tag.html](examples/link-tag.html) | Discovery tag snippet |

---

## Reference implementation

[agenticfeed.ai](https://agenticfeed.ai) is the hosted reference implementation of this standard.

It provides:

- Automatic product catalogue import from Shopify, WooCommerce, Google Merchant Center, and website crawling
- AI-generated intent data (questions, problems, use cases) per product
- Daily stock and price synchronisation
- Automatic discovery tag injection into Shopify themes via OAuth
- UTM-tagged product URLs for attribution tracking
- A merchant dashboard for managing feeds and subscriptions
- Agent Query API at `/agent/query` for natural language product matching
- A2A-compatible Agent Card at `/.well-known/agent.json`

Merchants who use agenticfeed.ai get a fully conformant agentic feed without writing any code. Developers who want to query product data by buyer intent can use the Agent Query API with their own Anthropic key.

---

## Contributing

Issues and pull requests are welcome at [github.com/bluestratus/agenticfeed](https://github.com/bluestratus/agenticfeed).

The goal of this standard is to be minimal and stable. Proposals that add complexity without clear benefit to AI agents or merchants will not be merged. The best contributions are real-world implementation experience, edge cases, and corrections.

---

## Licence

Published under the [MIT Licence](LICENSE). Anyone is free to implement compatible agentic feeds using this standard without restriction.
