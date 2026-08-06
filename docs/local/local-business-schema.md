# LocalBusiness schema graph

Ship **one** sitewide `LocalBusiness` node with `"@id": "#business"`, injected into `<head>` on every page, and make every per-page `Service` graph point `provider` at it by reference. That single node is what turns a pile of disconnected JSON-LD blocks into one confident entity that Google and answer engines can resolve — and it is the node that fixes the most common validator warning on local sites: `provider` references that resolve to nothing. This chapter covers the sitewide node, the per-page graphs that hang off it, NAP consistency, and the validation workflow that proves it all actually renders.

## The graph at a glance

| Node | Where it lives | Key point |
|---|---|---|
| `LocalBusiness` `@id=#business` | ONE sitewide block, in `<head>` on every page | Canonical business identity: name, address, geo, telephone, url, hours, `sameAs`. Everything else references it. **Never carries `aggregateRating` or `review`.** |
| `Organization` + `WebSite` | Usually CMS-auto, sitewide | Leave these to the platform if it emits them. Do not author a second `Organization`. |
| `Service` | Per service page and per city page | `provider: {"@id": "#business"}` links back to the sitewide node. This is where [ratings and reviews go](reviews.md). |
| `FAQPage` | Per page with a visible FAQ | Built deterministically from rendered Q&A — [next chapter](faq-schema.md). |
| `BreadcrumbList` | Per deep page | Mirrors the visible breadcrumb trail. Still renders in Google results. |
| `Person` (owner) | About page | `worksFor: {"@id": "#business"}` — an E-E-A-T anchor for bylines. |

Three or more schema types per page, cross-linked by `@id`, was the operational bar in the case-study engagement — industry research (as of 2026-07, reported) put rich structured data in ~61% of ChatGPT-cited pages vs ~25% of ordinary URLs.

## The sitewide `#business` node

The template — replace every value with your real, publicly published business data:

```json
{
  "@context": "https://schema.org",
  "@type": ["LandscapingBusiness", "LocalBusiness"],
  "@id": "https://www.example.com/#business",
  "name": "Example Outdoor Services",
  "description": "Family-owned lawn care, landscaping, and snow removal serving Exampleville and nearby cities since 2020.",
  "url": "https://www.example.com",
  "image": "https://www.example.com/images/social-default.jpg",
  "logo": "https://www.example.com/images/logo.png",
  "telephone": "+1-555-0134",
  "email": "hello@example.com",
  "priceRange": "$$",
  "foundingDate": "2020",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Exampleville",
    "addressRegion": "MN",
    "postalCode": "55000",
    "addressCountry": "US"
  },
  "geo": { "@type": "GeoCoordinates", "latitude": 44.79, "longitude": -93.52 },
  "areaServed": [
    { "@type": "City", "name": "Exampleville" },
    { "@type": "City", "name": "Neighborville" }
  ],
  "openingHoursSpecification": [{
    "@type": "OpeningHoursSpecification",
    "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
    "opens": "07:00",
    "closes": "19:00"
  }],
  "sameAs": [
    "https://www.facebook.com/examplebusiness/",
    "https://www.instagram.com/examplebusiness/"
  ]
}
```

Notes that matter:

- **Pick the most specific `@type` you honestly qualify for** (`LandscapingBusiness`, `Plumber`, `Electrician`…) and pair it with plain `LocalBusiness` in an array, as above.
- **Use an absolute `@id`** (`https://domain/#business`), not a bare fragment. Per-page references can still say `{"@id": "#business"}` on the same page, but the absolute form survives copy-paste across templates and makes domain-migration sweeps greppable. If you migrate domains, sweep every `@id` — the case site had to update all of them on cutover day.
- **`sameAs` is your corroboration graph.** Facebook, Instagram, Yelp, BBB — plus `hasMap` with your Google Maps listing URL once the Business Profile exists. This is how the website entity and the GBP entity get tied together (shipped on the case site 2026-07-18, verified in the rendered head).
- **No `aggregateRating`, no `review`, ever, on this node.** Google's December 2025 policy makes self-serving ratings on `LocalBusiness`/`Organization` ineligible for rich results and a manual-action risk. Stars go on a `Service` node — the [reviews chapter](reviews.md) covers this in full.

## Why one sitewide node — the dangling-reference problem

Per-page Service graphs commonly emit:

```json
"provider": { "@id": "#business" }
```

That is a *reference*, not a definition. If nothing on the page defines `@id: "#business"`, the reference dangles: validators warn, and — worse — engines cannot consolidate your pages into one entity. This is not hypothetical. On the case site's first audit (2026-07-11), the Shakopee city page was already shipping a `Service` block pointing `provider` at `#business`, and **no page anywhere defined that node**. The homepage carried two duplicate `Organization` blocks (one on the legacy `http://schema.org` context with a different logo URL and no `@id`) and zero `LocalBusiness`.

The fix is structural, and you make it once: inject the `#business` node into the `<head>` template that every page inherits. Every existing and future `provider` reference resolves from that moment on. Shipped and verified live the same night — the previously dangling city-page reference resolved without touching the city page at all.

## Per-page graphs — Service + BreadcrumbList

Each service page and each city page carries its own `@graph` in one `<script type="application/ld+json">` block:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Service",
      "@id": "https://www.example.com/service-areas/exampleville#service",
      "name": "Lawn Care & Landscaping in Exampleville, MN",
      "serviceType": "Lawn care service",
      "provider": { "@id": "https://www.example.com/#business" },
      "areaServed": {
        "@type": "City",
        "name": "Exampleville",
        "sameAs": "https://en.wikipedia.org/wiki/Exampleville,_Minnesota"
      },
      "url": "https://www.example.com/service-areas/exampleville",
      "description": "Unique, page-specific description."
    },
    {
      "@type": "BreadcrumbList",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://www.example.com/" },
        { "@type": "ListItem", "position": 2, "name": "Service areas", "item": "https://www.example.com/service-areas" },
        { "@type": "ListItem", "position": 3, "name": "Exampleville" }
      ]
    }
  ]
}
```

- **The Wikipedia `sameAs` on the `City` object is the entity-disambiguation trick** — it tells engines exactly *which* Exampleville you mean. Cheap, unambiguous, shipped on all seven city pages in the case.
- On service pages, `areaServed` lists all the cities you genuinely serve; on a city page it is that one city.
- `BreadcrumbList` mirrors the *visible* trail — do not mark up a navigation path the page does not show.
- Add the page's `FAQPage` node to the same `@graph` when the page has a visible FAQ ([next chapter](faq-schema.md)).

In the case engagement this pattern went onto all 15 service pages plus 7 city pages in one session, giving every page 3+ cross-linked types: `LocalBusiness` (sitewide) + `Service` + `BreadcrumbList` + `FAQPage`, alongside the platform's auto `Organization`/`WebSite`.

## One author per node — the CMS-auto vs hand-authored boundary

Most CMSes auto-emit `Organization` and `WebSite` JSON-LD. The rule that keeps validators and entity consolidation sane: **every node type has exactly one author.** Keep the platform's auto `Organization` + `WebSite`; hand-author `LocalBusiness`, `Service`, `FAQPage`, `BreadcrumbList`. Never inject a second `Organization` "just to be safe" — the case site's duplicate Organization blocks (different logo URLs, mixed `http`/`https` contexts) are exactly the kind of conflict that confuses entity consolidation, and merging them was step one of the schema work.

If your platform *cannot* emit LocalBusiness and you *can* hand-author, you own that node. If a platform update later starts auto-emitting it, turn one side off. Two blocks describing the same entity with different values is worse than either alone.

## NAP consistency — the field that spans every layer

Your **N**ame, **A**ddress, and **P**hone must be byte-identical across: the site footer, the `#business` node, Google Business Profile, and every directory listing. Not "close" — identical, down to suite numbers and formatting, because matching is done by machines. Two findings from the engagement research (2026-07, reported) make this concrete:

- AI assistants pull NAP from a narrow trusted set (Yelp, BBB, data aggregators). A stale number there means the AI recites a wrong contact *even if your site is perfect*.
- Businesses consistent across ~20 directories were ~3x more likely to appear in AI local recommendations.

And one finding from the field, the hard way: decide the canonical NAP **before** creating any listing. The case business had a site phone number and a different personal cell on old marketing collateral — that conflict had to be resolved before the citation program could start, because correcting it afterwards means re-editing 50 profiles by hand.

## Shipping and verifying

1. **Ground-truth the live site first.** Fetch each key page and extract every JSON-LD block:
   ```bash
   curl -s https://www.example.com/ \
     | python3 -c 'import sys,re,json; [print(json.loads(m).get("@type"), json.loads(m).get("@id")) for m in re.findall(r"<script type=\"application/ld\+json\">(.*?)</script>", sys.stdin.read(), re.S)]'
   ```
   List every node, every `@id` defined, every `@id` referenced. Dangling references and duplicate Organizations show up immediately.
2. **Inject the `#business` node** into the head-level layout template so it ships on every page.
3. **Add the per-page `@graph`** (Service + BreadcrumbList + FAQPage) to each service and city page.
4. **Validate** each page type in Google's Rich Results Test *and* the Schema Markup Validator (validator.schema.org). Rich Results Test only shows types Google renders; the validator catches structural problems in everything else.
5. **Fetch the live public page again and re-extract** — not the CMS preview, the real URL a crawler sees. Confirm the node actually renders and every reference resolves. On the case site, a "deployed" LocalBusiness block had been invisible for weeks because a deactivated template fork was shadowing the active one; only a live fetch caught it.
6. **Re-verify after every domain change or template edit**, on the public host. Rendered HTML is the only truth.

## Gotchas

- **The block that renders isn't the block you edited.** CMSes with copy-on-write template forks (Odoo and family) can leave your edit in a view that never renders. Verify by fetching the live page, never by trusting the editor. See [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md).
- **Schema behind a bot wall is invisible.** If a WAF challenge returns 429 to crawlers, your markup does not exist to them. Diagnose crawlability first — [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md).
- **Embedding JSON-LD in XML templates needs escaping.** In QWeb/XML-based CMSes, serialize with ASCII-safe JSON, escape `&`, `<`, `>`, and parse-validate the whole template as XML before pushing — one unescaped `&` in a URL truncated a live page section in the case engagement.
- **Absolute `@id`s go stale on migration.** Sweep every `@id` and URL in every schema block when the domain changes; the case migration re-emitted 32 templates' worth.
- **Do not mark up what the page doesn't show.** Hours, address, and reviews in schema must match visible content — schema-vs-visible drift is a policy problem, not just a bug ([FAQ chapter](faq-schema.md) shows the deterministic fix).
- **`areaServed` is a claim.** Only list cities you actually serve — the [service-areas chapter](service-areas.md) covers what happens when that claim gets audited.

## Related

- [FAQ schema from visible content](faq-schema.md) — the third node in each page's graph
- [Reviews — real ones only](reviews.md) — why ratings live on the Service node, and how to keep them true
- [Service-area pages](service-areas.md) — the pages these graphs ship on
- [Structured data (schema.org)](../google/structured-data.md) — JSON-LD and `@graph` fundamentals
- [Rich results](../google/rich-results.md) — which node types Google actually renders
- [Google Business Profile](../google/business-profile.md) — the other half of NAP consistency
- Source skill: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema)
