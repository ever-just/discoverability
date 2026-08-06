# Templates

Eight copy-paste starters for the artifacts this book tells you to ship. Every value is a placeholder (`example.com`, `YOUR-NAME`) — replace all of them before publishing. Three rules apply to everything on this page:

1. **Markup must mirror visible content.** Prices, FAQs, and reviews in schema must exactly match what the page shows — that's Google policy, not preference.
2. **Validate before and after deploy.** JSON-LD through the [Schema Markup Validator](https://validator.schema.org) and [Rich Results Test](https://search.google.com/test/rich-results), then `curl` the live page and confirm the block actually renders — database-verified is not rendered-verified.
3. **Never fabricate.** No invented reviews, ratings, or stats. An empty property is safe; a fake one risks a manual action.

## 1. LocalBusiness sitewide `@graph` (`@id: "#business"`)

Inject once into `<head>` on **every** page of the site. This is the canonical business identity node — per-page Service graphs (templates 3 and 7) reference it with `"provider": {"@id": "#business"}`, and this node is what resolves those references. Swap `LocalBusiness` for the closest subtype (`LandscapingBusiness`, `Plumber`, `Dentist`, …). Do **not** put `aggregateRating` or `review` here — see template 7.

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": ["LocalBusiness", "HomeAndConstructionBusiness"],
      "@id": "#business",
      "name": "YOUR-BUSINESS-NAME",
      "url": "https://example.com",
      "image": "https://example.com/images/storefront.jpg",
      "logo": "https://example.com/images/logo.png",
      "telephone": "+1-555-555-0100",
      "email": "hello@example.com",
      "priceRange": "$$",
      "foundingDate": "2020",
      "address": {
        "@type": "PostalAddress",
        "streetAddress": "123 Main St",
        "addressLocality": "YOUR-CITY",
        "addressRegion": "YOUR-STATE",
        "postalCode": "00000",
        "addressCountry": "US"
      },
      "geo": {
        "@type": "GeoCoordinates",
        "latitude": "44.0000",
        "longitude": "-93.0000"
      },
      "areaServed": [
        { "@type": "City", "name": "YOUR-CITY" },
        { "@type": "City", "name": "NEIGHBOR-CITY" }
      ],
      "openingHoursSpecification": [
        {
          "@type": "OpeningHoursSpecification",
          "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
          "opens": "07:00",
          "closes": "19:00"
        }
      ],
      "hasMap": "https://maps.google.com/?cid=YOUR-MAPS-CID",
      "sameAs": [
        "https://www.facebook.com/YOUR-NAME",
        "https://www.instagram.com/YOUR-NAME",
        "https://www.yelp.com/biz/YOUR-NAME"
      ]
    }
  ]
}
```

Full reasoning: [LocalBusiness schema graph](../local/local-business-schema.md).

## 2. SaaS dual-node `@graph` (SoftwareApplication + Service)

The two-node model for software products: `SoftwareApplication` is the only node Google renders a software rich result for, but `hasOfferCatalog` is only valid on Organization/Person/**Service** — so the pricing catalog hangs off a `Service` node, cross-linked by `@id`. Note the three Offer shapes: a free tier gets `price: "0"`, a paid tier gets a `UnitPriceSpecification` with `billingDuration: "P1M"` (per month), and a custom-priced tier gets **no price node at all** — `"0"` on it would read as "free".

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Organization",
      "@id": "https://example.com/#organization",
      "name": "YOUR-PRODUCT",
      "url": "https://example.com",
      "logo": {
        "@type": "ImageObject",
        "url": "https://example.com/images/logo.png"
      },
      "sameAs": [
        "https://github.com/YOUR-ORG",
        "https://www.linkedin.com/company/YOUR-NAME"
      ]
    },
    {
      "@type": ["SoftwareApplication", "WebApplication"],
      "@id": "https://example.com/#software",
      "name": "YOUR-PRODUCT",
      "url": "https://example.com",
      "applicationCategory": "DeveloperApplication",
      "operatingSystem": "Web",
      "publisher": { "@id": "https://example.com/#organization" },
      "featureList": [
        "FEATURE-ONE, stated as a plain capability",
        "FEATURE-TWO, stated as a plain capability",
        "FEATURE-THREE, stated as a plain capability"
      ],
      "offers": {
        "@type": "AggregateOffer",
        "lowPrice": "0",
        "highPrice": "99",
        "priceCurrency": "USD",
        "offerCount": "3"
      }
    },
    {
      "@type": "Service",
      "@id": "https://example.com/#service",
      "name": "YOUR-PRODUCT",
      "serviceType": "YOUR-CATEGORY-PHRASE",
      "provider": { "@id": "https://example.com/#organization" },
      "mainEntityOfPage": { "@id": "https://example.com/#software" },
      "hasOfferCatalog": {
        "@type": "OfferCatalog",
        "@id": "https://example.com/pricing#catalog",
        "name": "YOUR-PRODUCT plans",
        "itemListElement": [
          {
            "@type": "Offer",
            "name": "Free",
            "price": "0",
            "priceCurrency": "USD"
          },
          {
            "@type": "Offer",
            "name": "Pro",
            "priceSpecification": {
              "@type": "UnitPriceSpecification",
              "price": "99",
              "priceCurrency": "USD",
              "billingDuration": "P1M"
            }
          },
          {
            "@type": "Offer",
            "name": "Enterprise"
          }
        ]
      }
    }
  ]
}
```

Every price here must equal the price shown on the page. Worked example and node-by-node reasoning: [Rich results](../google/rich-results.md).

## 3. FAQPage

Only questions and answers that are **visible on the page** go in — Google requires it, and building the block by extracting the rendered Q&A (rather than authoring it by hand) is what keeps schema and content from drifting. Google dropped FAQ rich results from most SERPs, but Bing and AI answer engines still ingest FAQPage, which is why it stays worth shipping (as of 2026).

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "FIRST-QUESTION, worded exactly as it appears on the page?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "The first answer, verbatim from the visible page content."
      }
    },
    {
      "@type": "Question",
      "name": "SECOND-QUESTION, worded exactly as it appears on the page?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "The second answer, verbatim from the visible page content."
      }
    }
  ]
}
```

Extraction method (regex over both accordion and h3/p markups): [FAQ schema from visible content](../local/faq-schema.md).

## 4. robots.txt with the AI-crawler allow-list

Answer engines can only cite what their bots can fetch. The explicit per-bot groups make your intent unambiguous and survive a later careless `Disallow` edit to the `*` group. Add your own `Disallow` lines under `User-agent: *` for cart/session/admin paths; keep the `Sitemap:` line pointing at the real, canonical-host sitemap. (What each token controls: [AI crawler registry](crawler-registry.md).)

```text
# Default: everyone may crawl.
User-agent: *
Allow: /

# AI answer engines are welcome to read and cite our content.
User-agent: GPTBot
Allow: /

User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Claude-SearchBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /

Sitemap: https://example.com/sitemap.xml
```

A perfect robots.txt is worthless behind a bot challenge — verify with the curl pattern in the [crawler registry](crawler-registry.md) that these bots actually get a 200.

## 5. llms.txt (minimal)

The llmstxt.org format: one H1, one blockquote of grounding facts, H2 sections of annotated absolute links. Keep expectations calibrated — ~97% of llms.txt files get zero bot requests and Google says it doesn't use the file; the real consumers are coding agents, and the `## For AI agents` section is what points an LLM reading your human site at your machine surface. Ship it cheap; never instead of crawlability or schema.

```text
# YOUR-NAME

> Two or three sentences of grounding facts: what YOUR-NAME is, who it
> serves, where it operates, and the numbers that matter (founded YYYY,
> N.N-star rating from N reviews). Accurate and current — nothing invented.

## Services

- [Service one](https://example.com/services/service-one): one-line factual description
- [Service two](https://example.com/services/service-two): one-line factual description

## Key pages

- [Pricing](https://example.com/pricing): plans and current prices
- [About](https://example.com/about): who runs YOUR-NAME
- [Contact](https://example.com/contact): how to reach us

## For AI agents

- [Agent manifest](https://example.com/.well-known/agent/mcp.json): MCP endpoint, transport, and auth
- [API reference](https://example.com/docs/api): REST API with OpenAPI spec
```

The honest cost/benefit: [llms.txt — the reality check](../ai-search/llms-txt.md).

## 6. server.json for the MCP Registry

The one metadata record every MCP directory inherits. `name` is a reverse-DNS namespace for a domain you can prove you own (the `mcp-publisher` CLI walks you through the DNS TXT verification — publish exactly the value it prints). Keep `description` under 100 characters and intent-phrased: the verbs an agent's task would contain, not a slogan. The `remotes[].url` must stay byte-identical to your capability manifest and OAuth `resource` — endpoint drift across surfaces is the hardest-to-spot failure in the agent layer.

```json
{
  "name": "com.example/mcp",
  "description": "Create, connect, and manage YOUR-CATEGORY-NOUNS over MCP.",
  "repository": {
    "url": "https://github.com/YOUR-ORG/YOUR-REPO",
    "source": "github"
  },
  "version": "1.0.0",
  "remotes": [
    {
      "type": "streamable-http",
      "url": "https://mcp.example.com/mcp"
    }
  ]
}
```

Validate against the current schema in [modelcontextprotocol/registry](https://github.com/modelcontextprotocol/registry) (it evolves); locally-installed servers declare `packages` instead of `remotes`. Full publish flow: [The MCP Registry](../agents/mcp-registry.md).

## 7. Review markup on a Service node (Dec-2025 policy-compliant)

Per Google's December 2025 policy, `aggregateRating`/`review` on the LocalBusiness or Organization node is a **self-serving rating — ineligible for rich results and a manual-action risk**. Stars belong on a `Service` (or Product) node the business provides. Only real reviews: a real author name and the verbatim body, both visible on the page this block ships on. The `provider` reference resolves to template 1's sitewide `#business` node.

```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "YOUR-SERVICE-NAME",
  "serviceType": "YOUR-SERVICE-CATEGORY",
  "provider": { "@id": "#business" },
  "areaServed": [
    { "@type": "City", "name": "YOUR-CITY" }
  ],
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.9",
    "reviewCount": "37",
    "bestRating": "5"
  },
  "review": [
    {
      "@type": "Review",
      "author": { "@type": "Person", "name": "REAL-REVIEWER-NAME" },
      "reviewRating": { "@type": "Rating", "ratingValue": "5" },
      "reviewBody": "The verbatim review text, exactly as the customer wrote it and exactly as displayed on this page."
    }
  ]
}
```

Keep `ratingValue`/`reviewCount` synced to the live source (they drift within days when hardcoded). Policy detail and sync patterns: [Reviews — real ones only](../local/reviews.md).

## 8. The Ask-AI deep-link row

Five `<a href>` links that open the visitor's own AI assistant with a prompt about you pre-filled — user-initiated, so it's a legitimate CTA, not injection. Two hard rules: the prompt must be **self-contained** (bake 2–4 sentences of accurate product facts into it — do not depend on the LLM fetching your site, because it will search instead and may find nothing), and it must be URL-encoded (`encodeURIComponent`), with any literal `&` in an HTML attribute written as `&amp;` (Google's URL below). Schemes verified as of 2026-07 and undocumented — they break without notice, so re-verify quarterly. Gemini and Copilot are skipped deliberately: their `?q=` parameters don't work as of 2026.

```html
<div class="ask-ai">
  <span>Ask AI about us:</span>
  <a href="https://chatgpt.com/?q=YOUR-URL-ENCODED-PROMPT"
     target="_blank" rel="noopener noreferrer"
     aria-label="Ask ChatGPT about YOUR-NAME">ChatGPT</a>
  <a href="https://claude.ai/new?q=YOUR-URL-ENCODED-PROMPT"
     target="_blank" rel="noopener noreferrer"
     aria-label="Ask Claude about YOUR-NAME">Claude</a>
  <a href="https://www.perplexity.ai/search?q=YOUR-URL-ENCODED-PROMPT"
     target="_blank" rel="noopener noreferrer"
     aria-label="Ask Perplexity about YOUR-NAME">Perplexity</a>
  <a href="https://www.google.com/search?udm=50&amp;q=YOUR-URL-ENCODED-PROMPT"
     target="_blank" rel="noopener noreferrer"
     aria-label="Ask Google AI Mode about YOUR-NAME">Google AI Mode</a>
  <a href="https://grok.com/?q=YOUR-URL-ENCODED-PROMPT"
     target="_blank" rel="noopener noreferrer"
     aria-label="Ask Grok about YOUR-NAME">Grok</a>
</div>
```

Prompt shape that works: `Explain YOUR-NAME (example.com) in plain English. Grounding facts: [2–4 accurate sentences — what it does, who it serves]. Tell me what it does, who it is for, and how it works. More at https://example.com`. Per-provider behavior and icon sourcing: [The Ask-AI widget](../ai-search/ask-ai-widget.md).

## Related

- [Structured data (schema.org)](../google/structured-data.md) — the `@graph`/`@id` pattern behind templates 1, 2, and 7
- [Rich results](../google/rich-results.md) — which of these nodes Google actually renders
- [Reviews — real ones only](../local/reviews.md) — why template 7 is shaped the way it is
- [Sitemaps and robots.txt](../google/sitemaps-and-robots.md) — the rest of the robots story
- [AI crawler registry](crawler-registry.md) — the tokens in template 4, bot by bot
- [The MCP Registry](../agents/mcp-registry.md) — publishing template 6
- [The Ask-AI widget](../ai-search/ask-ai-widget.md) — template 8 in depth
