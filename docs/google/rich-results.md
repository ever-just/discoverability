# Rich results

**Google renders rich results only from specific node types — most schema.org markup is invisible in the SERP, and that's fine.** The craft has two halves: knowing which node types actually produce visual enhancements (and modeling your graph so the *right* node carries the right data), and staying eligible under policies that now explicitly punish self-serving markup. The worked example running through this chapter is the customdomain.ai dual-node graph — designed, shipped, and validated "eligible" in Google's own test within ~30 hours in July 2026.

## What Google actually renders (as of 2026-08)

| Node type | SERP rendering | Status | Notes |
|---|---|---|---|
| `SoftwareApplication` / `WebApplication` | Software card (name, category, price/offer, rating) | **Live** | The *only* node Google renders a software rich result from — a `Service` or `Product` describing your software won't trigger it. `applicationCategory` must be one of Google's ~25 supported values (e.g. `DeveloperApplication`). |
| `BreadcrumbList` | Breadcrumb trail replacing the raw URL | **Live** | Cheap, still renders, ship it everywhere deep. |
| `Product` + offers/reviews | Price, availability, stars | **Live** | For merchandise; SaaS pricing belongs on Offer/OfferCatalog structures instead (below). |
| `JobPosting` | Google Jobs surface | **Live** | Strict required fields — see the repair recipe below. |
| `Review`/`AggregateRating` (on eligible types) | Star snippets | **Live, restricted** | The Dec-2025 self-serving rule decides *where* these may sit — see below. |
| `FAQPage` | FAQ accordions in the SERP | **Dead for most sites** | Google restricted FAQ rich results to a handful of authority sites in 2023 and our mid-2026 research pass recorded them as fully retired. **Keep the markup anyway**: Bing and AI answer engines still ingest FAQPage, and FAQ-structured pages correlated with ~3x AI citations (reported). Just don't promise SERP stars from it. |
| `Organization`, `WebSite`, `Service`, `OfferCatalog`, `DefinedTermSet` | *No visual rich result* | — | Still valuable: they feed the Knowledge Graph, entity resolution, and AI engines. Absence from the Rich Results Test report is expected, not failure. |

The strategic reading: **rich-result eligibility is a modeling constraint, not a decoration.** You place data on the node Google renders, then cross-link the rest of the graph around it.

## The dual-node pattern (worked example)

The problem, on customdomain.ai: a SaaS product wants (a) the software rich result and (b) machine-readable tiered pricing. Two vocabulary facts collide:

1. Google renders software cards **only** from `SoftwareApplication`/`WebApplication` — so that node must exist and carry the product identity.
2. `hasOfferCatalog` — the property that models a tiered pricing catalog — is **only valid on `Organization`, `Person`, and `Service`. Not on `SoftwareApplication`.**

So neither node can do both jobs. The design that shipped (July 2026, live on the homepage and pricing page):

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Organization", "@id": "https://customdomain.ai/#organization",
      "sameAs": ["https://github.com/ever-just"] },
    { "@type": ["SoftwareApplication", "WebApplication"],
      "@id": "https://customdomain.ai/#software",
      "applicationCategory": "DeveloperApplication",
      "featureList": ["automatic DNS configuration", "on-demand TLS", "..."],
      "offers": { "@type": "AggregateOffer", "lowPrice": "0", "highPrice": "749",
                  "priceCurrency": "USD" } },
    { "@type": "Service", "@id": "https://customdomain.ai/#service",
      "provider": { "@id": "https://customdomain.ai/#organization" },
      "hasOfferCatalog": { "@id": "https://customdomain.ai/pricing#catalog" },
      "mainEntityOfPage": { "@id": "https://customdomain.ai/#software" } }
  ]
}
```

- The **`SoftwareApplication`** node carries the rich-result payload: category, an 18-item `featureList` (the densest "what it does" surface AI readers get), and an `AggregateOffer` price range.
- The **`Service`** node carries `hasOfferCatalog` pointing at the pricing page's catalog, plus `availableChannel` entries advertising the REST API and MCP endpoint.
- They cross-link by `@id` (`mainEntityOfPage`, shared `provider`/`seller`), so consumers see one product, two facets — not two products.
- `Product` was deliberately **omitted**: redundant overlap with SoftwareApplication would have created a second competing identity.

This is the general lesson: when a property you need is invalid on the node you need rendered, **split into two cross-linked nodes** rather than forcing invalid markup (validators will flag it; consumers will drop it).

## Pricing: OfferCatalog and UnitPriceSpecification

The pricing page's catalog, condensed from the shipped block:

```json
{ "@type": "OfferCatalog", "@id": "https://customdomain.ai/pricing#catalog",
  "itemListElement": [
    { "@type": "Offer", "name": "Starter", "price": "0", "priceCurrency": "USD" },
    { "@type": "Offer", "name": "Startup",
      "priceSpecification": { "@type": "UnitPriceSpecification",
        "price": "249", "priceCurrency": "USD",
        "billingIncrement": 1, "unitCode": "MON" } },
    { "@type": "Offer", "name": "Enterprise" }
  ],
  "seller": { "@id": "https://customdomain.ai/#organization" } }
```

Three rules, all policy- or ambiguity-driven:

- **Recurring prices use `UnitPriceSpecification`**, not a bare `price` — a bare `"price": "249"` doesn't say *per what*; the price specification carries the monthly recurrence so $249/mo can't be read as one-time.
- **Custom-priced tiers get NO price node at all.** `"price": "0"` on an Enterprise tier reads as *free* — the worst possible parse. Name the Offer and stop.
- **Markup prices must equal on-page prices** (structured-data policy). Treat the pricing page as the source of truth and regenerate markup when it changes.

## Review stars and the self-serving rule

**Per Google's December 2025 policy restatement: self-serving `aggregateRating`/`review` markup on your own `LocalBusiness` or `Organization` node is ineligible for rich results and risks a manual action.** Stars belong on a thing you *provide* — a `Service` or `Product` node — backed by reviews actually displayed on the page.

Shipped-and-validated pattern (local-business case): one `Service` node on the reviews page carrying `aggregateRating {4.9, reviewCount}` plus an array of `Review` nodes — each a real displayed review, `Person` author, **verbatim** body — with `provider` pointing at the sitewide `#business` node, which itself carries **no rating markup**. Never fabricate or paraphrase reviews, and omit `aggregateRating` entirely until real reviews are on-page: an invented rating to unlock the software card is a spam violation, not a growth hack. Full policy + live-sync mechanics in [Reviews — real ones only](../local/reviews.md).

## JobPosting: the strict one

JobPosting rich results fail loudly on missing required fields. Real repair (found via [URL Inspection](search-console.md#url-inspection-api-the-ground-truth-diagnostic): indexed pages, rich results FAILED with `Missing field "description"`): wrap the rendered job body in the description field, complete the `PostalAddress`, set a rolling `validThrough` (+60 days), add `baseSalary` as a `MonetaryAmount`, and a `TELECOMMUTE` fallback for remote roles. Validate in the Rich Results Test, then push the fixed URLs through the Indexing API — JobPosting is one of the [two types it legitimately serves](search-console.md#the-indexing-apis-real-scope).

## Eligibility and manual-action risk

Rich results are a privilege with published rules; the ones that actually bite:

- **Markup must match visible content** — prices, FAQs, reviews, ratings. Mismatch is where "spammy structured markup" manual actions come from, and a manual action suppresses rich results (or worse) site-wide.
- **Self-serving ratings** (Dec-2025 rule above).
- **Schema describing fabricated things** — the severest class we've audited: `ImageObject` markup declaring an AI-generated hero image to be a photograph of a real job. Structured data is a *claims surface*; audit it like one ([authenticity audits](../local/authenticity.md)).
- **Eligibility ≠ guarantee.** "Eligible" in the test means Google *may* render it, subject to query, quality, and trust — a young domain can be eligible for months before stars appear. Don't ship more markup to force it; build [the entity](../foundations/entities-and-trust.md).

## Validation and monitoring

1. **[Rich Results Test](https://search.google.com/test/rich-results)** per URL — remember it reports only visually-rendered types. The shipped example returned: homepage "3 valid items — eligible" (Organization ×2, Software Apps ×1; the only notes were the intentionally-omitted `aggregateRating`), pricing "2 valid items" (Breadcrumbs, Organization), zero errors. That's what done looks like.
2. **[Schema Markup Validator](https://validator.schema.org/)** for everything the RRT ignores (Service, OfferCatalog, DefinedTermSet…).
3. **Live-page `@type` census** ([the workflow](structured-data.md#the-validation-workflow)) — confirm the shipped graph is the rendered graph.
4. **GSC monitoring**: the per-type enhancement reports under *Shopping/Enhancements* track valid/invalid item counts over time; [URL Inspection](search-console.md#url-inspection-api-the-ground-truth-diagnostic) gives per-URL rich-result verdicts with field-level errors. Re-check after every template change — a template bug breaks *every* page of that type at once.

## Beyond schema: SERP presentation

Your result's look is more than markup (all shipped-and-verified, mid-2026):

- **Favicon**: Google wants ≥48×48px. Gotcha that cost a client their branding: **Chrome prefers an SVG favicon over ICO/PNG regardless of declaration order** — a platform-injected SVG icon hijacked the tenant's SERP favicon until the SVG itself was replaced. Ship a proper multi-size ICO *and* control any SVG icon link; crop marks for legibility at 16px (a full wordmark is mush). Google re-crawls favicons on its own schedule — expect days-to-weeks of lag after fixing.
- **Title**: ~60 characters of display width; front-load the unique keyword part. Audited pages ran 71–78 chars with a long "| Brand Name" suffix truncating — shorten the suffix, not the keywords.
- **Meta description**: ~150–160 chars, embed the live trust signal (rating/count — dynamically, so it can't drift) and locale terms.

## Gotchas

- **Forcing invalid property/type combinations** instead of splitting nodes — the `hasOfferCatalog`-on-SoftwareApplication mistake; validators flag it, consumers drop it.
- **`price: "0"` on custom tiers** — reads as free. Omit the price node.
- **Ratings on the business/Organization node** — ineligible + manual-action risk since Dec 2025.
- **Chasing FAQ stars in Google** — dead for most sites; the markup's remaining value is Bing + AI ingestion.
- **Reading RRT absence as failure** for non-rendered types — use validator.schema.org for those.
- **Validating once and never monitoring** — template regressions silently zero out an enhancement type; GSC's counts are your alarm.
- **Fabricating anything to unlock a rich result** — ratings, reviews, images-as-photos. This is the one gotcha that ends in a manual action instead of a lost opportunity.

## Related

- [Structured data](structured-data.md) — graph fundamentals, @id linking, and the validation workflow this chapter builds on
- [Search Console](search-console.md) — URL Inspection diagnostics and the Indexing API's legitimate JobPosting use
- [Local: Reviews](../local/reviews.md) — the Dec-2025 rule in full, plus live review syncing
- [Local: Authenticity audits](../local/authenticity.md) — schema as a claims surface
- [Case study: customdomain.ai](../case-studies/customdomain-ai.md) — the dual-node graph in its full context
- [Appendix: templates](../appendix/templates.md) — the SaaS dual-node starter

Source skills: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema) · [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization) · [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo)
