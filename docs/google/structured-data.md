# Structured data (schema.org)

**Structured data is how machines parse what you are — and in 2026 it feeds three consumers at once: Google's rich results, the Knowledge Graph, and AI answer engines.** The craft is not "add some JSON-LD"; it's building one coherent, cross-linked entity graph that exactly mirrors your visible content, shipping it without fighting your CMS's auto-generated blocks, and verifying what actually rendered — because the gap between "schema in the database" and "schema in the served HTML" has eaten weeks of work.

## JSON-LD fundamentals

Use JSON-LD (not microdata or RDFa — Google's stated preference, and the only format that's sane to generate programmatically). A block is a `<script type="application/ld+json">` element, valid anywhere in the page but conventionally in `<head>` or at the end of the content:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "@id": "https://example.com/#organization",
  "name": "Example Co",
  "url": "https://example.com/",
  "logo": { "@type": "ImageObject", "url": "https://example.com/logo.png" },
  "sameAs": [
    "https://github.com/example",
    "https://www.linkedin.com/company/example"
  ]
}
</script>
```

Ground rules that carry everything else:

- **`https://schema.org`** as context (one legacy `http://` block on a page we audited created a duplicate-entity mess).
- **Every entity gets a stable `@id`** — an absolute URL + fragment (`https://example.com/#organization`). The `@id` is the entity's name across pages and blocks; without it, consumers can't merge nodes.
- **Schema describes what's on the page.** Google's structured-data policy requires markup to reflect visible content — prices in markup must equal prices on the page, FAQ answers must be visible, reviews must actually be displayed. This isn't just policy: divergence is how markup drifts into lies.
- The vocabulary is versioned and living (schema.org v30.0 as of March 2026) — when in doubt about what a property is valid *on*, check the type page, not your intuition. A single vocabulary constraint (`hasOfferCatalog` not valid on `SoftwareApplication`) forced the entire [dual-node design](rich-results.md#the-dual-node-pattern-worked-example) in our flagship deployment.

## The `@graph` + `@id` cross-linking pattern

Real pages need several entities. Ship them as **one block with a `@graph` array**, cross-linked by `@id`, rather than a pile of independent blocks:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Organization", "@id": "https://example.com/#organization", "name": "Example Co" },
    { "@type": "WebSite", "@id": "https://example.com/#website",
      "publisher": { "@id": "https://example.com/#organization" } },
    { "@type": "Service", "@id": "https://example.com/#service",
      "provider": { "@id": "https://example.com/#organization" } }
  ]
}
```

Why this matters:

- **References resolve to one confident entity.** `"provider": {"@id": "...#organization"}` is a *reference* — consumers resolve it against the node that defines that `@id`. Research we relied on for the local-business program: rich schema appeared in ~61% of ChatGPT-cited pages vs ~25% baseline, and cross-linked multi-type graphs correlate with more citations (reported figures, 2026) — AI engines reward being able to resolve one entity instead of guessing among fragments.
- **Dangling references are a real bug class.** We inherited a site whose per-page `Service` blocks all pointed at `{"@id": "#business"}` — and nothing anywhere defined `#business`. Validators warn; consumers shrug and drop the relation. The fix is the **sitewide anchor node** pattern: inject the one canonical node (`LocalBusiness` with `@id: #business`, or your `#organization`) into `<head>` on *every* page, so every page's references resolve locally. Full local-business treatment in [LocalBusiness schema graph](../local/local-business-schema.md).
- **Cross-page identity**: use the same absolute `@id`s on every page. The homepage's `#organization` and the pricing page's `seller: {"@id": ...#organization}` should be byte-identical strings.

## CMS-auto vs hand-authored — never both on the same node

Most platforms auto-emit some JSON-LD (typically `Organization` + `WebSite`, generated from site settings). You will usually *also* want hand-authored, page-specific graphs. The collision rule, learned the hard way:

- **Never emit two competing definitions of the same node type.** We found pages carrying two conflicting `Organization` blocks — one with email+logo, one with alternateName+`@id` — which "can confuse entity consolidation." Consolidate to one, or make both emitters share the same `@id` so consumers merge instead of forking the entity.
- **Divide by node type.** The stable division that works: let the platform own `Organization` + `WebSite` (and *feed* it — the auto block's `sameAs` is typically driven by the site's social-profile fields, so filling those fields IS your schema work), and hand-author everything the platform can't know: `LocalBusiness`/`Service`/`FAQPage`/`SoftwareApplication`/`OfferCatalog`/`BreadcrumbList`.
- **Know your generator's failure mode.** The platform generator we work with wraps itself in try/except and emits *empty markup on any error* — a bad value fails silently. "No error" proves nothing; only fetching the rendered page proves the block exists.

## Deterministic generation from visible content

The strongest practice in this chapter: **generate schema *from* the rendered page, don't author it *about* the page.** Hand-authored markup drifts the moment someone edits copy; extracted markup can't.

- **FAQPage**: parse the page's real, visible Q&A (handle both your accordion markup and plain `h3`/`p` pairs), strip tags, and emit exactly those pairs. If a Q&A isn't visible on the page, it does not go in the schema — that's Google policy *and* your anti-drift device. Regenerate whenever the visible FAQ changes. (Recipe in [FAQ schema from visible content](../local/faq-schema.md).)
- **Reviews**: only reviews actually displayed, author + verbatim body, aggregate numbers matching the visible count — and mind the [placement policy](rich-results.md#review-stars-and-the-self-serving-rule).
- **Prices**: markup prices must equal page prices. We caught a site whose FAQ schema effectively told Google prices "don't exist" while the pages showed conflicting numbers — price consistency is a schema artifact, not just a copy problem.
- **Counts and ratings that change** (review counts, ratings): drive both the visible text and the schema from one stored value updated by one job, or they *will* diverge — ours drifted within days (47 vs 48 vs 51 across pages) until a single source of truth patched every surface.

The build pattern that shipped our largest graph: a generator script assembles the `@graph` from a product-facts file, an offline validation harness re-parses the JSON, counts `@type`s, and checks every Offer's price shape — all **before** anything touches the site.

## The validation workflow

Four stages, in order. Each catches a failure class the previous one can't.

1. **Parse it yourself, offline.** `python -c "import json,sys; json.load(open('block.json'))"` plus your own shape checks. Catches syntax and structural bugs in seconds instead of after deploy.
2. **[Rich Results Test](https://search.google.com/test/rich-results)** — Google's view. It reports **only the types Google renders visually** (an eligibility check, not a parser). Our shipped homepage returned "3 valid items" (Organization, Software App) with the absence of `Service`/`OfferCatalog`/`DefinedTermSet` from the report being *expected, not failure* — see [Rich results](rich-results.md).
3. **[Schema Markup Validator](https://validator.schema.org/)** — the full vocabulary parse. This is where dangling `@id`s, invalid property/type combinations, and the non-rich-result nodes get checked.
4. **Fetch the LIVE page and parse the JSON-LD you actually shipped.** The stage everyone skips and the only one that's ground truth:

```bash
curl -s https://example.com/ \
  | grep -o '"@type": *"[A-Za-z]*"' | sort | uniq -c   # the @type census
```

Compare the census against what you deployed. **DB-verified ≠ rendered-verified.** The war story that mandates this stage: a schema block was written to the homepage view, verified present in the database, reported "done" — and never rendered, because the site's homepage route was configured to serve a *different* page's view than the one the tooling mapped to `/`. The diagnosis (byte-identical HTML across fetches → not a CDN → survived restart → prove-which-template-renders-the-route with a distinctive string) took longer than the entire schema design. **Prove which template serves the route before you trust any write to it**, and re-run the census after every deploy that touches templates.

Two adjacent serving traps: WYSIWYG editors commonly **strip `<script>` tags on save** — never "just re-save the page" to flush a cache with JSON-LD in the body; and on template-cached CMSs, view-level writes may not render until workers restart (content-field writes render immediately). Details in [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md).

## Gotchas

- **Dangling `@id` references** — every `{"@id": ...}` you emit must be defined by a node somewhere on that page (the sitewide anchor pattern makes this automatic).
- **Two Organization definitions** from CMS-auto + hand-authored blocks — consolidate or share the `@id`.
- **Inventing content to enrich markup** — FAQs nobody can see, ratings with no visible reviews, prices not on the page: policy violations and [manual-action risk](rich-results.md#eligibility-and-manual-action-risk).
- **A lone `sameAs` is weak entity evidence.** One GitHub URL gives answer engines almost no corroboration graph; build out consistent profiles (LinkedIn, Crunchbase, Wikidata, directories) with identical name/logo/description — see [Entities and trust](../foundations/entities-and-trust.md).
- **Validating the test page, shipping something else** — the Rich Results Test on a URL tests what Google fetches *now*; if your CDN/proxy caches HTML, you may validate stale content in either direction. Purge, then validate.
- **Schema on an uncrawlable page is a no-op.** A WAF challenge or robots block means no consumer ever sees your graph — [crawlability first](sitemaps-and-robots.md), always.
- **Editor sanitizers strip script tags** — inject JSON-LD via the template/view layer, not through the visual editor.

## Related

- [Rich results](rich-results.md) — which node types Google renders, and the dual-node worked example
- [Local: LocalBusiness schema graph](../local/local-business-schema.md) — the sitewide `#business` anchor pattern in full
- [Local: FAQ schema from visible content](../local/faq-schema.md) — the deterministic extractor
- [Local: Reviews](../local/reviews.md) — real-review policy and live syncing
- [Foundations: entities and trust](../foundations/entities-and-trust.md) — sameAs corroboration and E-E-A-T
- [Technical: reverse proxies and CMS traps](../technical/reverse-proxy-cms.md) — why rendered-verification is non-negotiable
- [Appendix: templates](../appendix/templates.md) — copy-paste graph starters

Source skills: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema) · [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo) · [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization)
