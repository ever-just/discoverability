# Reviews — real ones only

Star ratings are the highest-value structured data a local business can ship and the easiest to ship illegally. Two rules govern everything here: **`review` and `aggregateRating` belong on a `Service` node, never on your `LocalBusiness` or `Organization` node**, and **every review you mark up must be a real review with a real author and its verbatim body, rendered on the page**. This chapter covers the December 2025 policy, what "real" means in practice, the live-sync architecture that keeps a displayed rating honest across dozens of surfaces, the FTC exposure of fabricated reviews, and how to ask for reviews without breaking anyone's rules.

## The December 2025 policy, in full

Google's restatement (December 2025, as read and acted on during the case engagement in July 2026 — verify current wording before you rely on it) tightened the long-standing self-serving-review restriction:

- **Self-serving `aggregateRating` and `review` on the business entity — `LocalBusiness`, `Organization`, and their subtypes — is ineligible for rich results.**
- Worse than ineligible: it is a **manual-action risk**, and a manual action is sitewide, not per-page.
- "Self-serving" means reviews *about the business itself* that the business collects, curates, and displays on its own property. That is exactly what a testimonials page is.

The reasoning is simple once you see it: everyone would give themselves five stars. Google's position is that **the business entity's stars natively belong on the Google Business Profile**, where Google controls collection.

### Where the stars go instead

Reviews of a **thing the business provides** — a `Service`, or a `Product` — remain eligible. So the sanctioned shape is:

```json
{
  "@type": "Service",
  "@id": "https://www.example.com/reviews#service",
  "name": "Lawn Care, Landscaping & Snow Removal",
  "serviceType": "Lawn care service",
  "provider": { "@id": "https://www.example.com/#business" },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.9",
    "reviewCount": "52",
    "bestRating": "5",
    "worstRating": "1"
  },
  "review": [
    {
      "@type": "Review",
      "author": { "@type": "Person", "name": "A real reviewer's published name" },
      "reviewRating": { "@type": "Rating", "ratingValue": "5" },
      "reviewBody": "The verbatim text of the review, exactly as displayed on the page."
    }
  ]
}
```

The `provider` reference points back at the sitewide `#business` node ([schema graph chapter](local-business-schema.md)), so the entity stays connected without the business node ever carrying a rating.

**What we shipped:** on headsupoutdoorservices.com (2026-07-11) this went live as a single `Service` node on the reviews page carrying a 4.9/52 aggregate plus 35 `Review` children — one for each real, named review the page actually rendered — generated programmatically from the rendered HTML rather than typed by hand. It validated clean and no manual action followed. Whether it ever produced a visible rich result is **unmeasured**; the reason to do it correctly is eligibility and AI extractability, not a promised star in the SERP.

!!! warning "The common trap"
    A hardcoded "★ 4.9 · 52 reviews" badge in your header is *page chrome*, not schema — that is fine. The violation is putting `aggregateRating` inside the `LocalBusiness` JSON-LD. Sites acquire this by copying a template. Grep your rendered pages for `aggregateRating` and check which node it sits inside.

## What counts as a real review

Four tests, all of which must pass:

| Test | Meaning |
|---|---|
| **Real author** | A person who actually left it, under the name they published it as. A city-only attribution ("— Google review, Shakopee") with no author is a fabricated testimonial. Never invent one. |
| **Verbatim body** | The exact text, not a tightened or "improved" version. Paraphrasing for polish makes it a quote you wrote. |
| **Rendered on the page** | Markup must mirror visible content. Do not mark up reviews you display nowhere. |
| **Aggregate matches reality** | `ratingValue` and `reviewCount` equal the true totals, and equal what the page displays. |

On the aggregate-vs-detail gap: it is legitimate for `reviewCount` to exceed the number of `Review` nodes — 52 total reviews, 35 shown and marked up — **provided the page itself displays the 52 figure and the figure is true**. What is not legitimate is an aggregate you rounded up, or one that has silently gone stale.

That staleness is the whole reason the next section exists.

## Live-syncing Google reviews

A rating printed into templates is true on the day it is typed and false soon after. On the case site, "4.9 · 51 Google reviews" was hardcoded in dozens of places — up to roughly 92 occurrences across about 60 templates — and had already forked: different pages showed 47, 48, and 51 at the same time, and the reviews page's SEO title said 47 while its body said 51. Every one of those is a small honesty failure and a schema-vs-visible mismatch.

### The architecture

```
Google Places API (max 5 reviews per call)
        │  rating + total review count + 5 newest reviews
        ▼
   sync job  ── runs OUTSIDE the CMS, every 6h ──►  dedupe on Google's review id
        │                                            accumulate into a stored model
        ▼
  ONE stored source of truth (config params + review records)
        │
        ├─► page templates render the number dynamically
        ├─► aria-labels render it dynamically
        ├─► JSON-LD ratingValue / reviewCount
        └─► patcher rewrites the surfaces that CANNOT be dynamic
              (SEO meta titles/descriptions stored as plain DB fields)
```

Four constraints shaped that design, each learned the hard way:

1. **The Places API returns at most 5 reviews.** Verified across keys and both API versions (2026-07). You cannot pull all 51 from it. Accumulating across pulls grows the set going forward but **cannot backfill** older reviews.
2. **The Business Profile API is the only official route to all of a business's reviews** — and it is OAuth-only (API keys can never reach it) and ships at **effective quota 0** until Google approves an access application. The signature is `429 RESOURCE_EXHAUSTED` with an empty quota bucket. That is *pending approval*, not rate limiting: **do not retry**. Apply early; see [Google Business Profile](../google/business-profile.md).
3. **The sync cannot run inside a sandboxed CMS.** The case platform's server-action sandbox rejects code that so much as names `__import__` — no sockets, no HTTP, ever. The sync script lives outside the CMS (a box cron or scheduled job) and writes back through the API.
4. **Some surfaces are not templatable.** A `<title>` cannot be a template expression, and SEO meta fields are usually separate database columns that view edits never touch. Those get *patched* by the sync job. In the case engagement that one non-templatable title was the single surface that silently drifted after everything else was fixed.

### Finding your place ID when Google can't find you

The sync needs a place ID. Getting it exposed a bigger problem: **Places text search returned zero results** for the case business by name, by name+city, by address, and by phone, with a 30 km location bias — while direct competitors resolved instantly. That pattern means an unverified, unpublished, or suspended Business Profile: the business was invisible in Google local search and Maps regardless of anything on the website.

The place ID was recovered from the site's own "leave us a review" link, which carries `writereview?placeid=…` in the URL. If your own search returns nothing, treat it as a profile emergency, not a data-plumbing problem — the map pack is worth more than every on-site optimization combined.

### Rolling it out without breaking the site

- **Never blanket find-and-replace a rating string.** "4.9" also occurs inside SVG path data (where it becomes `4.919`) and inside attribute values. Match display *phrases*, and skip anything between `<` and `>`.
- **Sweep the rendered page, not the templates.** Counts hide in meta descriptions, `aria-label` attributes, alt text, and JSON-LD — surfaces a template grep misses. The case rollout converted about 60 views and 35 aria-labels to dynamic values, then verified zero leaked template expressions on the rendered output.
- **Expect stale aliases.** Two different rating parameter keys had accreted on the case site (one referenced far more widely than the other); the sync writes both. Check your alias map against live data on every run.
- **Only bust caches when the value actually changed** — a sync that restarts or invalidates on every run is a self-inflicted availability problem.

**How you know it worked:** fetch every page that repeats the number, extract the rating and count from body text, `aria-label`s, meta description, and JSON-LD, and assert one distinct value per field across the whole site. Run it after each sync. The first run of the case pipeline caught the count had already drifted 47→48 the same day it shipped.

## Fabricated reviews are a legal problem, not just an SEO one

The US FTC's rule on consumer reviews and testimonials (16 CFR Part 465, effective October 2024) makes fake and misattributed reviews a civil-penalty matter. The exposure compounds when the site *also* claims its reviews are verified — that turns an unattributable pull-quote into a deceptive claim about the claim.

That is not theoretical either. The case site's service-area pages carried seven testimonials that matched **no** customer record and **no** published review anywhere, on a site whose reviews page asserted every review was verified. All seven were replaced with verbatim real Google reviews. The rule that came out of it: **only a reviewer already published with a city may be city-attributed on a city page** — and on that site only 4 of 35 reviews named a city, which capped how much per-city social proof could honestly exist. If the honest number is small, the honest answer is a smaller section, not a fuller invented one.

Related false-claim exposure from the same engagement: a "BBB Accredited" badge on a site whose own linked BBB profile said *not accredited*. The business genuinely held an A+ rating, so the honest fix was to say that. See [Authenticity audits](authenticity.md).

## How to solicit reviews properly

Volume of real reviews is the actual lever — industry research the engagement relied on (2026-07, reported) puts an ~4.3-star threshold on AI assistants' willingness to recommend a local business, and reviews at roughly 20% of local-pack weight. Get them like this:

- **Ship a one-click write-review deep link.** Google's `search.google.com/local/writereview?placeid=…` opens the review form directly. Put it on the reviews page *and* the homepage. Check where it points: on the case site a "Read all reviews on Google" link actually opened the *write-review* form — a mislabeled CTA that silently cost real review reads.
- **Ask by email, timed.** Post-job requests convert best while the work is fresh: hours after a quick visit, a day or two after a large install. Research cited in the engagement (reported) puts re-engagement after a month at roughly 11%, so late asks are near-worthless. The case site sent a request 21 days after a job closed as won.
- **Ask everyone, not just happy customers.** Filtering who gets asked by expected sentiment ("review gating") violates Google's policies and is squarely within the FTC rule's scope. Ask all of them.
- **Respond to reviews** — response rate is an explicit signal, with >80% cited as the bar in the engagement research (reported).
- **Never solicit Yelp reviews.** Yelp's "don't ask" policy demotes businesses that do. Claim the listing, keep the NAP right, and let it accumulate.
- **Never buy, trade, or write reviews.** Beyond the FTC exposure, it makes every other trust signal you ship worthless.

## Verifying the whole thing

1. **Fetch the live public page** and extract all JSON-LD. Confirm `aggregateRating` appears on the `Service` node and appears **nowhere** on `LocalBusiness`/`Organization`.
2. **Count `Review` nodes and compare to reviews rendered on the page.** Equal, or the markup is claiming something the page does not show.
3. **Spot-check three `reviewBody` values against the visible text** character for character.
4. **Run the one-distinct-value check** across body, aria-labels, meta fields, and JSON-LD.
5. **Validate** in the Schema Markup Validator (structure) and Rich Results Test (eligibility). Expect the Rich Results Test to be quiet — a validated `Service` node with reviews is the goal, a rendered star is a bonus.
6. **Re-run after every sync and every content edit.**

## Gotchas

- **`aggregateRating` on the business node.** The single most common local-schema violation, and the most expensive one — sitewide manual-action risk. Move it to `Service`.
- **Marking up reviews the page doesn't render.** Same class of violation as invented FAQ answers; both are schema-vs-visible drift.
- **Rounded or aspirational aggregates.** A 4.87 displayed as 4.9 is fine only if that is what the source actually reports and what the page displays. Inventing a count is not a rounding decision.
- **`429 RESOURCE_EXHAUSTED` from the Business Profile API is not rate limiting.** Empty quota bucket = your access application is pending. Retrying wastes days.
- **Places API caps at 5 reviews forever.** No key, no version, no plan changes that. Design for accumulation.
- **CMS sandboxes block outbound HTTP.** If your sync "works in testing" but never runs in production, check whether the platform's automation sandbox permits network calls at all.
- **A rating string inside SVG path data.** Blanket replacement corrupts artwork and aria-labels. Match phrases, not digits.
- **The stale surface you forgot.** SEO meta title/description live outside the template. Include them in every sweep.
- **An invisible Business Profile makes the rating unverifiable.** If Places search cannot find you, neither can anyone else — fix the profile before optimizing the markup.

## Related

- [LocalBusiness schema graph](local-business-schema.md) — the node that must never carry a rating
- [FAQ schema from visible content](faq-schema.md) — the same deterministic anti-drift discipline
- [Service-area pages](service-areas.md) — per-city review attribution rules
- [Authenticity audits](authenticity.md) — finding fabricated testimonials before a regulator does
- [Google Business Profile](../google/business-profile.md) — where the stars natively belong, and the API-quota reality
- [Rich results](../google/rich-results.md) — eligibility policies across node types
- [Off-site signals](../ai-search/offsite-signals.md) — the review platforms answer engines actually read
- Source skills: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema), [marketing-site-authenticity-audit](https://github.com/ever-just/agentskills/tree/main/skills/marketing-site-authenticity-audit)
