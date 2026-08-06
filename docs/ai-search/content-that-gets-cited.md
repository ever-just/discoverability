# Content that gets cited

**AI engines cite pages they can extract a clean answer from. That means: a direct 40–60-word answer before anything else, question-shaped H2s that mirror real queries, at least one HTML table, evidence with dates and sources, and a topical cluster wired hub-and-spoke so engines see you as the authority on the category — with content-page schema mirroring what's visible.** This chapter is the full recipe: page structure, cluster architecture, schema, and the freshness mechanics that get new pages seen.

The evidence this recipe rests on (external research, compiled 2026-07, unless marked): comparison tables extracted **~81% vs ~23%** for prose; FAQ-structured pages earning **~3x** citations; **~44%** of ChatGPT citations coming from the first 30% of a page; the Princeton GEO study's **+40%** from statistics, **+40%** from authoritative citations, **+28%** from quotations; rich schema on **~61%** of ChatGPT-cited pages vs ~25% baseline.

## The answer-first page shape

Every page in the program follows the same skeleton. It works because it matches how engines chunk, rank, and lift content.

1. **BLUF (bottom line up front).** The first paragraph answers the target query directly in ~40–60 words, before any preamble. This is the single highest-leverage structural move — it's what an engine lifts as *the* answer. No "in this guide we'll explore…", no brand throat-clearing.
2. **Exactly one H1.** Everything else is H2/H3. (Cheap to check mechanically: the page source should contain `<h1` exactly once.)
3. **Question-shaped H2s** that mirror how people phrase the query to an AI: "What is on-demand TLS?", "How much does snow removal cost per visit?". Each H2's first 1–2 sentences answer the question; detail follows. Sections must be **self-contained** — no "as mentioned above", because an engine may lift the section without its neighbors.
4. **At least one HTML `<table>`** carrying the decision-relevant contrast: build-vs-buy, feature/price, per-option comparison. Put the facts you most want quoted in the table, not around it.
5. **Stats, dates, and named sources.** "Responding to a quote within 5 minutes vs 30 can lift contact rates 20x (source)" beats "fast responses matter". Cite genuine authorities relevant to your field — a university extension guide, a standards body, a government dataset — not content-farm roundups.
6. **A short visible FAQ** (4–12 real buyer questions) with matching `FAQPage` schema — *only* when the Q&A actually renders on the page (see [schema rules](#content-page-schema-the-additive-layer) below).
7. **Plain ending.** No keyword-stuffed footer paragraphs. Google explicitly names AI-specific rewriting and tiny-chunk fragmentation as ineffective (documented).

### Writing definitions AI quotes verbatim

Glossary and definition content deserves special care because engines lift definitions word-for-word. The pattern:

- **One liftable sentence.** The first sentence of the page *is* the definition — subject, category, differentiator — and matches the H1 and the `DefinedTerm` schema description byte-for-byte. Three copies of the same sentence (visible BLUF, schema, meta description) is how you make an engine confident.
- **Own the category vocabulary.** If the category name is contested ("custom domains as a service" vs "managed custom domains"), publish the one-sentence definition for *each* phrasing and state your canonical term. Whoever publishes the definition gets quoted as its author — this is how a small site becomes the reference for a term (measured on the customdomain.ai glossary, 2026-07).
- **Noun + recognized job.** Coin terms as *known-thing + what-it-does* ("on-demand TLS", "bring your own domain") rather than pure neologisms an engine has no context for.

## Topical clusters: pillar + glossary + how-to

One page rarely earns category authority. A **cluster** — one comprehensive pillar (the hub) plus glossary definitions and how-to guides (the spokes) — is the unit that does. The shape we shipped on customdomain.ai (measured, 2026-07: pillar of ~3,000 words with 11 question-shaped H2s and a 12-question FAQ, 4 glossary definitions, 3 guides, one index page — all live and in the sitemap within days):

| Content type | Length | Structure | Per-page JSON-LD | Links |
|---|---|---|---|---|
| **Pillar** (hub) | ~3,000–5,000 words | One H1, ~10 question-shaped H2s, a build-vs-buy comparison table, a "key terms, defined" block linking every glossary entry | `FAQPage` + a product type (e.g. `SoftwareApplication`) | OUT to every spoke |
| **Glossary definition** (spoke) | ~1,000–1,400 words | Answer-first BLUF definition, question H2s, short FAQ | `DefinedTerm` + `FAQPage` | Back to pillar + sibling definitions + one audience page |
| **How-to guide** (spoke) | ~1,000–1,600 words | Answer-first, numbered steps, short FAQ, **no fabricated APIs** — product specifics defer to real docs | `TechArticle` + `FAQPage` | Back to pillar + sibling guides + one audience page |
| **Glossary/hub index** | Short | A list hubbing the whole cluster | `CollectionPage` + `ItemList` | To every definition, guide, and the pillar |

Word counts are comprehensiveness minimums, not padding targets. If the honest treatment of a term is 700 words, stop at 700.

### Hub-and-spoke wiring (do all four)

The link graph is what earns topical authority and moves internal link equity. All four moves, every cluster:

1. **Pillar links out to every spoke** — a dedicated hub section ("Key terms, defined") rather than scattered inline links.
2. **Every spoke links back** to the pillar, to its sibling spokes, and to the single most relevant audience/landing page (that last link is what converts authority into pipeline).
3. **An index page** lists and links the entire cluster, carrying `CollectionPage` + `ItemList` schema that enumerates the cluster URLs.
4. **A sitewide footer link to the hub** (e.g. a "Resources" column leading with the pillar and glossary) so *every* page on the site passes equity into the cluster.

### Choosing what the cluster targets

Don't guess. Map real queries by awareness stage — problem-aware ("SSL cert stuck pending"), solution-aware ("custom domains as a service"), vendor-aware ("X alternatives"), dev/agent ("API to connect a domain") — and check who currently ranks and who AI currently cites for each. Prefer vacant or weakly-held queries over dream head terms. The full triage method, with the striking-distance mining loop, is in [Keyword and SERP strategy](../google/keyword-strategy.md).

## Content-page schema: the additive layer

Your site already (should) have a sitewide graph — `Organization`, `WebSite`, and whatever your [site type calls for](../google/structured-data.md). Content pages **add** a page-specific block of a *different* `@type`. Two rules before the types:

- **Never emit a second `Organization` or `WebSite` node.** If your CMS auto-injects them sitewide, a hand-authored duplicate creates two competing identity nodes and confuses entity consolidation. Additive means *different types only*.
- **Schema mirrors visible content.** Google requires FAQ answers to be visible on the page; the same discipline (schema as a machine-readable mirror of what renders, never an invention) applies to every type here. Build it *from* the rendered page, not from imagination.

The four content types:

```html
<!-- Glossary definition page -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "DefinedTerm",
  "name": "Bring your own domain",
  "description": "One sentence matching the on-page H1 and BLUF definition.",
  "inDefinedTermSet": "https://example.com/glossary"
}
</script>

<!-- Any page with a VISIBLE FAQ -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [{
    "@type": "Question",
    "name": "What is multi-tenant TLS?",
    "acceptedAnswer": { "@type": "Answer", "text": "The exact answer text that appears on the page." }
  }]
}
</script>

<!-- How-to guide -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "TechArticle",
  "headline": "How to add custom domains to your SaaS",
  "description": "One sentence matching the BLUF.",
  "author": { "@type": "Organization", "name": "Your Company" }
}
</script>

<!-- Cluster index page -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "CollectionPage",
  "name": "Glossary",
  "mainEntity": {
    "@type": "ItemList",
    "itemListElement": [
      { "@type": "ListItem", "position": 1, "url": "https://example.com/glossary/bring-your-own-domain" },
      { "@type": "ListItem", "position": 2, "url": "https://example.com/glossary/multi-tenant-tls" }
    ]
  }
}
</script>
```

A note on FAQ specifically: Google stopped showing FAQ *rich results* for most sites (restricted 2023; retirement of the remainder reported May 2026), but FAQ structure remains one of the strongest AI-extraction signals — FAQ-shaped content is weighted heavily in ChatGPT source selection and maps one-to-one to how people phrase prompts (external research, compiled 2026-07). Keep the visible FAQ + `FAQPage` pair; just don't expect Google stars for it.

## Freshness and the recrawl reality

Publishing the cluster is half the job; the other half is making sure engines *see* it. What's actually true, per engine (as of 2026-07):

- **Perplexity rewards recency** — fresh content gets recrawled in ~2–7 days and favored in answers (external research). Publish seasonal content *ahead* of its season; refresh cornerstone pages on a real cadence.
- **Bing acts on [IndexNow](../bing/indexnow.md)** — submit each new URL on publish; a scheduled sweep that diffs the sitemap and submits changes makes this automatic (measured: a 6-hour sweep + submit loop on customdomain.ai, 2026-07).
- **Google re-crawls the sitemap it already knows.** No ping, no force-index ([the myths](geo-fundamentals.md#ground-truth-the-facts-that-kill-the-myths)). Which makes sitemap correctness the whole Google-freshness game:
    - **`<lastmod>` must be truthful** — from the CMS's content-updated timestamp, never build time. Stamping every URL "now" on every deploy is fake freshness and erodes trust in the whole file.
    - **CMS-cached sitemaps hide new pages.** Several CMSes (Odoo among them) cache the rendered sitemap for hours; a page published today may be absent from `/sitemap.xml` until the cache clears — so neither Google nor your IndexNow sweep sees it. Clear or auto-expire the sitemap cache on publish. Details and the cron pattern: [Sitemaps and robots.txt](../google/sitemaps-and-robots.md), [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md).

## Verify before you call it shipped

1. **View the source, not the DevTools DOM.** `curl -s https://example.com/page | less` — is the BLUF in the raw HTML? Client-rendered content is invisible to most AI fetchers ([Rendering & WAFs](../technical/rendering-and-waf.md)).
2. **Validate every JSON-LD block** — parse it (any JSON linter), then run the page through validator.schema.org and Google's Rich Results Test. Expect the Rich Results Test to show only Google's visual types; absence of `DefinedTerm`/`CollectionPage` there is normal, not failure.
3. **Confirm the page is in the sitemap** — grep the full `<loc>https://example.com/page</loc>` line (full line, not a substring, so a wrong-host or partial match can't fool you).
4. **Confirm the internal links render** — the pillar's hub block, the spoke's back-links, the footer Resources link.
5. **Spot-check extraction** — ask ChatGPT and Perplexity the page's target question a week later and see whether your page appears in citations; log it in the [audit sheet](ai-visibility-audit.md).

## Gotchas

- **The marketing preamble.** Opening with brand story instead of the answer forfeits the first-30%-of-page window where ~44% of citations come from. BLUF or bust.
- **FAQ schema for invisible questions.** Bolting `FAQPage` onto a page with no rendered FAQ violates Google's visible-content requirement and risks the page's eligibility. Add the visible section first, then mirror it.
- **Duplicate identity nodes.** A hand-authored `Organization` block on a site whose CMS already injects one = two competing identities. Check what's auto-injected before adding anything (`curl -s URL | grep -o 'application/ld+json'` then read the blocks).
- **WYSIWYG editors that strip your schema.** Some site editors sanitize `<script type="application/ld+json">` out of the body on save (measured, 2026-07). After any visual edit, re-fetch the live page and confirm the block survived.
- **Fabricated specifics.** How-to guides that invent API endpoints, prices, or statistics to sound complete. Engines cross-check; buyers do too; and it violates the [honesty doctrine](../local/authenticity.md). Defer product specifics to the real docs and link them.
- **City-swap and template-swap cloning.** Generating near-identical pages per city/segment with nouns swapped is a doorway pattern Google's 2025 spam updates target — and it produced measurably duplicate content in our own audit (~72% cross-city similarity before rewrite; measured, 2026-07). Every page must survive being read *blind* — could a reader tell which city/segment it's for from the content alone?
- **Publishing into a cached sitemap.** The silent killer of cluster launches — everything live, nothing discoverable. Verify the `<loc>` lines, not the CMS admin screen.

## Related

- [GEO fundamentals](geo-fundamentals.md) — why this recipe, with the evidence
- [Keyword and SERP strategy](../google/keyword-strategy.md) — choosing targets the cluster can win
- [Structured data](../google/structured-data.md) — the sitewide graph this layer adds onto
- [Sitemaps and robots.txt](../google/sitemaps-and-robots.md) — the freshness plumbing
- [Auditing your AI visibility](ai-visibility-audit.md) — measuring whether the content gets cited
- Source skills: [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization), [everjust-website-geo-content](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-geo-content)
