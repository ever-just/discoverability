# Service-area pages

A city page has to earn its existence with evidence. The test is blunt: **if you cannot point to work you actually did there, customers you actually have there, or geography you can actually verify, the page is a doorway page** — and doorway pages are a documented Google spam policy, a liability in AI answers that cross-source, and in one real case a reputational emergency. This chapter covers the audit that decides which city pages live, the city-limits verification test for photos, per-city review attribution, Census-derived maps (and why Google Maps screenshots and drive times cannot be used), and the honest-coverage pattern for everywhere you serve but cannot prove.

## The teardown that motivates this chapter

The case site (headsupoutdoorservices.com, audited 2026-07) shipped **22 thin street- and neighborhood-level "service area" pages, linked from every footer**. What the audit found:

- The pages were near-identical. City-blinded similarity across the seven surviving city pages measured **72.4%**, with 3 of 10 content sections **byte-identical** between cities.
- Three pages advertised service on **sovereign tribal land** the business had no relationship with — handled first, as a reputational problem rather than an SEO one.
- **Zero of 242 leads** in the customer database were attributed to the service-area section. The pages produced no measurable business.
- Not one verified winter photo of the company's own work existed anywhere, while pages implied local seasonal work everywhere.

Outcome: 22 pages unlinked, 301-redirected into their real city pages, and deindexed. Seven city pages were kept and rebuilt on evidence. The traffic cost of removing them was **unmeasured** — they were producing nothing measurable to begin with.

The counter-example was considered and rejected explicitly: a national competitor runs roughly 5,900 programmatic location×service pages. That works as an aggregator playbook with an aggregator's link profile. For a single operator it is thin content at scale, and the August and December 2025 Google updates were aimed squarely at it.

## Does this city page deserve to exist?

Before designing anything, answer: **what is actually, verifiably different about this city?** There are only three honest sources of difference.

| Source | Strength | How you verify it |
|---|---|---|
| **Geography** | Full-strength, free, verifiable today | Municipal boundaries, rivers, highways, straight-line distance from your base. Public-domain data. Available for every city whether or not you have worked there. |
| **Operating reality** | True, sometimes unflattering | Leads and customers per city from your CRM. Real, but it will tell you some cities do not deserve a page. |
| **Work actually done there** | Strongest | Photos with provable location, reviews from that city, jobs on record. Requires provenance discipline (below). |

If a city offers you nothing but geography, you can still ship a *thin but honest* page — a real boundary map, a real distance, real services, real coverage statement — as long as you do not manufacture the other two. If a place offers you not even that (a street, a subdivision, a neighborhood name you cannot geocode), it does not get a page.

In the case rebuild, geography being the only full-strength differentiator available for every city is what made a real municipal-boundary map "the load-bearing element of the whole strategy."

### Weight the investment by real demand

Pull leads or customers per city from your CRM and let the numbers decide page depth. The case distribution was stark: 64 leads in the home city, 20 and 14 in the two adjacent ones, then 4, 3, 2, and 0 at the edges. A city with 64 leads earns a deep page with photos, reviews, and street-level detail. A city with 0 earns a coverage statement and a link.

## The city-limits verification test

This is the rule that killed most of the fake local proof on the case site, and it is the one thing to take away from this chapter:

> **A photo may carry a city claim only if its EXIF GPS coordinates geocode inside that city's actual municipal boundary.**

Two failures that make this non-optional:

- **Postal city ≠ municipal city.** One photo carried a mailing address in the city it was captioned with, but its coordinates geocoded to a *township* outside city limits. The caption had to say so.
- **A camera can stand in the next city.** A landmark photo was nearly captioned with the wrong city because the photographer's GPS position sat across the boundary from the landmark. Always geocode the *camera's* coordinates against city limits before captioning.

Of the case site's candidate photos, **only 3 had EXIF GPS at all, and only 2 survived the city-limits check.** That is the realistic yield. Plan for it.

```bash
# 1) does this image even carry coordinates?
exiftool -GPSLatitude -GPSLongitude -DateTimeOriginal photo.jpg

# 2) geocode the coordinates against real municipal boundaries, not postal data.
#    US Census TIGER "Places" are the authoritative municipal geometry and are public domain.
#    Point-in-polygon against the city's TIGER Place polygon = the answer.
```

Two registers, never blended:

- **WORK** — photos of your own jobs. City-captioned only when the coordinates prove it.
- **PLACE** — photos of the town itself, licensed (Creative Commons or purchased), rendered small and bordered, with the credit stored on the asset record and displayed, and captioned explicitly as *the town, not one of our jobs*.

Mixing those two registers is precisely how competitors fabricate local proof. Keeping them separate is cheap and defensible. See [Authenticity audits](authenticity.md) for the full audit method.

## Real per-city reviews

The rule: **only a reviewer whose published review already names the city may be attributed to that city.** You cannot infer a reviewer's town from your own records and print it next to their words — that publishes a customer's location.

The honest yield is small. On the case site only **4 of 35 reviews named a city**, which capped how much per-city social proof could exist. Seven testimonials that matched no customer record and no published review were removed entirely and replaced with verbatim real reviews — on a site that separately claimed all its reviews were verified, making those seven an FTC exposure as well as an SEO one. Full treatment in [Reviews — real ones only](reviews.md).

When a city has no attributable review, show the sitewide rating with its real total and no city claim. A smaller honest section beats a fuller invented one.

## Maps: use Census geometry, not Google Maps

Every instinct says "embed a Google map." Do not — for licensing reasons, not technical ones. As of 2026-07, from reading the Maps Platform terms during the case build:

- **Static Maps cannot draw municipal boundaries at all**, which is the one thing a service-area map needs to show.
- **Map attribution is legally immovable** — you may not restyle or reposition it to fit your design.
- **Google-measured drive times are Maps Content and may not be baked into your page.** "18 minutes from our shop" sourced from Maps is a terms violation, not a nice detail. Publish **straight-line miles** instead, computed from coordinates you own.

What we shipped instead: **an inline SVG built from US Census TIGER municipal geometry** — public domain, roughly 17 KB gzipped, zero external requests, $0 recurring. It beat the alternatives on every axis that mattered:

| Option | Verdict |
|---|---|
| TIGER-derived inline SVG | Public domain, no external requests, styleable with the site's theme, draws real boundaries. **Chosen.** |
| Google Static Maps | Cannot draw municipal boundaries; attribution immovable. |
| Google Maps JS API | Styling lives in a console, cannot follow the site's seasonal theme; heavy. |
| MapLibre + free OSM tiles | 248 KB of JavaScript, and the free tiles attach ODbL share-alike terms to your own artwork. |

Practical notes from shipping it: put the map CSS once in the site chrome, not per page; add a mobile media query or boundary labels render around 5px on phones; and drop marker rings that fall outside the viewBox rather than expanding the box.

## Anatomy of a city page that survives scrutiny

Ordered as it appears on the page:

1. **An answer-first opener** — what you do in this city, in two or three lifting-friendly sentences, in the first ~200 words.
2. **A real municipal-boundary map** (TIGER SVG) — the city's actual shape, not a generic pin.
3. **A straight-line distance line** — miles from a real anchor point. If your business record has no street address, anchor from a stated, honest reference (the case pages re-anchored from "our shop" to the middle of the home city once the company record proved to have no address on file).
4. **WORK photos** with city-limits-verified GPS, and/or a small credited **PLACE plate** labeled as the town.
5. **A verbatim real review**, city-attributed only if the published review names the city.
6. **Street or neighborhood detail only where it is real** — the case rule was a privacy floor of at least two customers on a street before naming it, and neighborhood names removed entirely when a geocoder could not confirm they exist.
7. **A ZIP-code coverage checker**, implemented once in the site chrome — with adjacent cities sharing a ZIP prefix explicitly excluded, or it will happily tell someone in the next town you serve them.
8. **A lead form** with a UTM medium identifying the city-page source, so the page's value becomes measurable.
9. **The page's `@graph`** — `Service` with `areaServed` set to that one `City` (plus a Wikipedia `sameAs` for disambiguation), `BreadcrumbList`, and an `FAQPage` built from the page's visible Q&A. See [the schema graph](local-business-schema.md) and [FAQ schema](faq-schema.md).

### Measure city pages without installing analytics

The case site had no analytics at all and a decision about consent tooling nobody wanted to make. The workaround: add a UTM source on the city-page lead forms so city-page leads land attributable in the CRM. It measures the thing that matters — leads — with no third-party dependency and no consent banner. It is also what proved the old doorway pages produced nothing.

## When a city page does *not* deserve to exist

Ship **honest coverage** instead:

- **One service-areas hub** listing every city, town, and township you genuinely serve, as plain text plus a coverage map.
- **The ZIP checker** on the hub, so "do you come to X?" is answerable without a page per X.
- **`areaServed` in your schema** listing every city — machine-readable coverage without a thin page for each.
- **A visually-hidden but real link list** if you need crawl paths to remain intact after removing pages. Real anchors, real destinations — this is an accessibility-neutral crawl path, not a cloaking trick.

Coverage claims cost nothing and carry no doorway risk. Pages carry both.

## The teardown procedure

1. **Inventory** every page under your service-area section, with its inbound internal links, its indexed status, and its lead count.
2. **Score similarity city-blinded** — strip the city names and diff. Sections that come out byte-identical are the doorway signature. Anything above roughly 70% similar needs rebuilding or removing.
3. **Classify**: keep and rebuild (real evidence exists) / merge (a street or neighborhood inside a city you do serve) / delete (a place you do not serve, or cannot verify).
4. **301 the merges and deletes** to their nearest true page — the city page, or the hub. Do not leave 404s where internal links pointed.
5. **Unlink them from the footer** in the same change. A sitewide footer link is what makes a thin page look like a network.
6. **Unpublish**, so they leave the sitemap.
7. **Check for duplicates at the root.** The case site had a stray root-level city URL duplicating its `/service-areas/<city>` page, both live and both indexed — 301 the stray and unpublish it.
8. **Verify**: fetch each old URL and confirm a 301 to the intended target; re-fetch the sitemap and confirm the removed URLs are gone; confirm the surviving pages still emit their full `@graph`.

## Gotchas

- **A footer link block is what turns thin pages into a doorway network.** Sitewide linking is the amplifier; removing the links is half the remediation.
- **Postal address ≠ city limits.** ZIP codes cross municipal boundaries constantly. Geocode against real boundary geometry.
- **One photo captioned as several cities** is the single most detectable fabrication on a service-area cluster — the case site's hero was one image standing in for seven cities. Hash your images ([Authenticity audits](authenticity.md)).
- **Invented neighborhood names** fail a geocoder check instantly. If it does not exist in a gazetteer, it does not go on the page.
- **Drive times from Maps are not yours to publish.** Straight-line miles, always.
- **Street lists are a privacy surface.** Naming a street with one customer on it identifies that customer. Floor it at two, or drop the section.
- **Coverage checkers over-claim by default.** Shared ZIP prefixes with neighboring cities need explicit exclusions.
- **Removing pages without redirects loses whatever equity they had** and produces 404s from your own old footer links in caches and third-party copies.
- **Heavy hero images per city add up.** The case rebuild cut one city hero from about 1 MB to 137 KB at display width with responsive sources — city clusters multiply every asset mistake by the number of cities.
- **Do not conflate "missing from the sitemap" with "not indexable."** They are separate problems with separate fixes ([Sitemaps and robots.txt](../google/sitemaps-and-robots.md)).

## Related

- [LocalBusiness schema graph](local-business-schema.md) — the `Service` + `BreadcrumbList` graph every city page carries
- [FAQ schema from visible content](faq-schema.md) — the third node on each city page
- [Reviews — real ones only](reviews.md) — per-city attribution rules and the FTC exposure
- [Authenticity audits](authenticity.md) — photo provenance and reused-image detection in full
- [Sitemaps and robots.txt](../google/sitemaps-and-robots.md) — deindexing and sitemap hygiene after a teardown
- [Keyword and SERP strategy](../google/keyword-strategy.md) — why long-tail local beats aggregator-owned head terms
- [Launch a local business](../playbooks/local-business.md) — where the service-area build slots into the sequence
- Source skills: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema), [marketing-site-authenticity-audit](https://github.com/ever-just/agentskills/tree/main/skills/marketing-site-authenticity-audit)
