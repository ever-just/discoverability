# Heads Up Outdoor Services — the local-business case study

headsupoutdoorservices.com is a family-owned lawn care, landscaping, and snow removal business serving Shakopee, MN and the southwest Twin Cities metro. In July 2026 its website was rebuilt on a multi-tenant CMS platform and taken through the full local discoverability stack: entity schema, policy-safe reviews, an API-driven Search Console setup, honest service-area pages, and an AI-crawler allow-list. This page is the dated record — anchored by two incidents that this book exists to help you avoid: a security setting that had erased the domain from search entirely, and a sitewide `noindex` that shipped silently for 11 days.

!!! abstract "What this proves"
    - **The catastrophic problems were infrastructure, not content.** A WAF challenge and a CMS domain-field trap each zeroed out visibility while the site looked perfect to humans. Both were invisible from inside the CMS; both were found only by testing the public surface the way crawlers see it.
    - **The local stack is an evidence discipline.** Every claim that survived — ratings, reviews, city pages, photos — traces to a verifiable source: a live API, a published review, EXIF GPS that survives a city-limits check. Everything that couldn't be verified was removed or relabeled.
    - **Google's plumbing is fully automatable — up to a line.** Property verification, sitemap submission, and diagnostics ran end-to-end through APIs. Account creation, CAPTCHAs, and business verification stayed human, and the record says so.
    - **Baselines make failure visible.** Pulling Search Console data programmatically is what turned "rankings feel slow" into "every page is noindexed" — in one query.

## The starting problem

Two sites, both losing. The brand's real domain ran on its previous hosting with an attack-challenge mode enabled — a security toggle doing exactly its job, against everyone. The rebuilt, crawlable site sat on a low-authority platform subdomain that no brand equity pointed at. The audit's framing became a book-wide principle: **the #1 SEO move is sometimes a decision, not code** — no on-site optimization mattered until the owned domain pointed at the crawlable site.

## Timeline

### 2026-07-09 — the premium rebuild

A multi-agent rebuild replaces a sprawling 62-page site with ~36 pages of new design and copy on the platform tenant, with per-page titles and meta descriptions across the board. First discoverability act: **22 thin street- and neighborhood-level "service area" pages are classified as doorway pages and removed via 301s** — a liability teardown, not a loss. A hard constraint is set on day one that shapes everything after: never invent facts, reviews, or photos.

### 2026-07-11 — the crawlability crisis, and the 90-minute schema program

The SEO engagement starts the way every engagement should: fetching the live site as `Googlebot`, `GPTBot`, and `PerplexityBot`.

!!! warning "The domain had been erased from search"
    Every request to the real .com — including robots.txt and sitemap.xml — returned **HTTP 429 with a challenge page** (the hosting platform's attack-challenge mode, fingerprinted by its response headers). No user-agent allowlist existed; non-rendering crawlers got challenge HTML with no title, no content, no schema. The result, verified the same hour: `site:headsupoutdoorservices.com` returned **zero indexed pages**, and the business's Facebook page outranked its own domain for its own name. A security toggle had removed the business from search and AI answers simultaneously — while the site worked fine in a browser. The fix was configuration (and ultimately migration), not SEO. Full pattern: [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md).

The same night, on the crawlable rebuild, the hand-authored AEO schema ships in about 90 minutes:

- A sitewide **LocalBusiness** node with a stable `@id`, injected once in the site chrome — which *resolved dangling references*: per-page Service blocks had been pointing `provider` at a `#business` node that didn't exist. ([LocalBusiness schema graph](../local/local-business-schema.md))
- **Reviews done to policy:** 35 real, on-page reviews marked up verbatim with the aggregate rating on a **Service node — never the business node** — per Google's Dec-2025 restatement that self-serving ratings on LocalBusiness/Organization are ineligible and risk manual action. ([Reviews — real ones only](../local/reviews.md))
- **FAQPage generated deterministically from each page's visible Q&A** (an extractor over the rendered markup, so schema can never drift from content), plus Service + BreadcrumbList graphs on all 15 service pages. ([FAQ schema from visible content](../local/faq-schema.md))
- A robots.txt that **explicitly welcomes AI crawlers** (GPTBot, OAI-SearchBot, ChatGPT-User, ClaudeBot, Claude-SearchBot, PerplexityBot, Google-Extended). ([AI crawlers](../ai-search/ai-crawlers.md))
- A stray duplicate city URL 301'd to its canonical.

### 2026-07-12 → 07-13 — knowledge capture and the honest counter-strategy

The session's methods are distilled into reusable skills, and a competitor teardown (a national gig-economy lawn platform) produces the honest countermoves: a published price list built only from prices already public on the site, cost-guide content doing real arithmetic on real ranges, and an explicit refusal to copy the competitor's ~5,900-page programmatic location farm — thin doorway spam for a single-crew operator.

### 2026-07-18 — the domain cutover and the API-driven Google wiring

The owner green-lights the domain move. The migration plan is adversarially audited *before* execution — the audit catches 12 blockers, three of which would have caused real damage (the registrar's DNS couldn't be pre-staged and would have gone live on a parked page; the platform proxy's catch-all would have served *another tenant's site* during propagation; the domain's transactional-email identity had been silently failed since May). Execution: pre-stage the full zone at a controllable DNS host, flip nameservers, pre-issue the TLS certificate via DNS validation *before* any traffic moves, add the proxy vhosts, cut the CMS over, and keep the platform machine host untouched — never redirect it; internal integrations depend on it. Rollback story: one DNS record flip. Full method: [Domain migrations](../technical/domain-migration.md).

Baked into the cutover checklist, because the platform's Host-rewriting proxy breaks CMS self-perception: a static robots.txt served at the proxy, sitemap rewriting to the public domain, and deletion of the CMS's cached sitemap. ([Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md))

The same night, Search Console is set up **entirely through APIs**: the Site Verification API mints a DNS TXT token, the record is written to the zone, the domain property is verified, the owner's own Google account is added as co-owner, the `sc-domain:` property is created, and the sitemap submitted — Google began fetching the same day. One follow-up on 07-19: the owner's own account still prompted for verification, because **each Google account verifying a domain property needs its own TXT token** — added alongside the first after reading the existing records so the SPF record survived the write. ([Google Search Console](../google/search-console.md))

### 2026-07-18 → 07-29 — live reviews: the count that kept drifting

The social proof ("4.9 stars, N Google reviews") was hardcoded in dozens of templates — and drifted against reality within days of every manual fix (47, 48, and 51 were live on different pages simultaneously at one point). The fix evolved into a pipeline:

- A sync job pulls the live rating and count from the Places API — which **hard-caps at 5 reviews per request on any key or version**; the only route to *all* reviews is the Business Profile API, which sat at effective quota zero pending Google's access approval (a documented state, not a bug — do not retry).
- A custom review store accumulates API pulls over time and seeds from the hand-verified on-page reviews.
- Every hardcoded instance (~60 templates, plus aria-labels and meta descriptions) becomes a dynamic render from **one stored value**; the one surface that can't render dynamically (the reviews page's meta title) is rewritten by the sync script itself.
- The sync runs *outside* the CMS — the platform's sandbox rejects any in-CMS code that could make HTTP calls.

En route, a finding that reframed the local program: **Places text search returned zero results for the business** across name, address, and phone queries while competitors resolved instantly — the Business Profile itself was effectively invisible in Maps and local search, pending owner verification. The listing ID was recovered from the site's own "leave us a review" link. ([Google Business Profile](../google/business-profile.md))

### 2026-07-29 — the noindex discovery

!!! warning "Eleven days of sitewide noindex, found by one API query"
    The first programmatic Search Console pull since migration showed a 90-day baseline of **24 clicks / 640 impressions / average position 14.5 — essentially all brand-name queries**. URL Inspection, page by page, returned the same verdict everywhere: *Excluded by 'noindex' tag*, with Google crawling daily and discarding every page. The live HTML confirmed `<meta name="robots" content="noindex">` sitewide. Root cause: the CMS computes indexability by comparing the request host to its configured domain field — and the platform proxy rewrites the Host header, so the comparison can never pass. Setting the domain field during the cutover had silently noindexed the entire site **from 07-18 to 07-29**. The fix was one field (clear it; canonicals fall back to the frozen base URL, which was correct throughout), verified live, sitemap resubmitted. Hours later a parallel work session *re-set* the same field to fix a cosmetic og:url mismatch and re-noindexed the site for a few seconds — proving the trap needed a platform-level fix, not tenant-level vigilance. The full bug class: [everjust-tenants](everjust-tenants.md).

The same day, the **service-area truthing** program rebuilds the city pages on verifiable evidence ([Service-area pages](../local/service-areas.md)):

- A 137-finding audit of the existing pages, including copy that was 72% identical across cities with the city name swapped.
- Municipal-boundary maps as inline SVG built from public-domain US Census TIGER geometry (~17 KB, zero external requests) — chosen over map-service embeds whose terms forbid boundary drawing and whose measured drive times may not be published as page content (straight-line miles instead).
- **The city-limits test:** a photo may carry a city claim only if its EXIF GPS survives a geocode against actual municipal boundaries. One photo's postal address said one city while its coordinates sat in a neighboring township; a landmark photo's camera position sat across a city line. Captions now say what's true.
- Testimonials that traced to no real customer or published review — some of them machine-generated in earlier build waves — replaced with **verbatim real Google reviews**, city-attributed only where the reviewer already published a city. The site claims its reviews are verified, which makes unverifiable testimonials a legal exposure, not just a quality issue. ([Authenticity audits](../local/authenticity.md))
- Page depth weighted by real CRM demand per city rather than uniform templates.

Housekeeping the same day: robots.txt was found blocking the site's own catalogue (`Disallow: /shop` boilerplate hiding 14 real, sellable service pages — unblocked, with only faceted query URLs and checkout kept blocked); the public host was found serving a *stale* robots.txt from a proxy cache that ignored no-store headers; and the homepage's absence from the sitemap was root-caused to a platform route decorator, with a validated patch staged for a maintenance window.

### 2026-07-30 — SERP branding and the citations program

Google's results were showing the *platform's* icon instead of the business's — the platform brand view injected an SVG favicon, and Chrome prefers SVG regardless of declaration order. Fixed per-tenant, along with the generator meta tag and a platform-credit link that had been rendering on the client site.

Then the **citations program**: ~50 local-citation platforms researched in parallel and triaged to 20 worth doing (12 more judged executable but worthless; 17 excluded as paid-only, dead, or requiring a home address to be published). Before any listing: the canonical-NAP decision, which surfaced real blockers — a phone-number conflict between the site and printed collateral, and no publishable street address. A claim-first sweep found essentially no existing listings to claim. And there the program stopped, honestly:

!!! note "The honest limit of agent automation"
    Every platform's research, forms, categories, and pre-written descriptions (at eight different character limits) were prepared by agents. **Zero listings were created** — account creation, passwords, SSO, and CAPTCHAs are deliberately human steps, and the handoff queue was the deliverable. The same audit also caught the site displaying a "BBB Accredited" badge while the bureau's own profile said otherwise — corrected to the accurate claim (an A+ *rating*, which the business does hold). Directory presence is an off-site trust surface; getting it wrong is worse than not having it. ([Off-site signals](../ai-search/offsite-signals.md))

## The numbers

| Metric | Value | Date | Status |
|---|---|---|---|
| Indexed pages on the .com, pre-fix | 0 (`site:` check; Facebook outranked the domain) | 2026-07-11 | Measured |
| Crawler responses on the old host | HTTP 429 + challenge for Googlebot, GPTBot, PerplexityBot — robots.txt and sitemap included | 2026-07-11 | Measured |
| GSC 90-day baseline (pre-noindex-fix) | 24 clicks / 640 impressions / 3.75% CTR / avg position 14.5 — brand queries only | 2026-07-29 | Measured |
| Sitewide noindex duration | 11 days (2026-07-18 → 07-29) | 2026-07-29 | Measured |
| Doorway pages removed | 22 (301 + deindex) | 2026-07-09 | Measured |
| Service pages with 3+ schema types | 15 | 2026-07-11 | Measured, live-verified |
| Reviews marked up verbatim on the Service node | 35 (of 51–52 on Google; 4.9 rating) | 2026-07-11 | Measured |
| Places API review cap | 5 per request, any key or version | 2026-07-29 | Measured (verified on two keys) |
| Review-count drift observed | 47 vs 48 vs 51 live simultaneously | 2026-07 (recurring) | Measured |
| Sitemap URL count over the window | 61 → 67 (cutover) → 82 (resubmit) → 93 | 2026-07 | Measured |
| Citation platforms researched → actionable → created | ~50 → 20 → **0** (blocked on account creation) | 2026-07-30 | Measured |
| Google fetch after API sitemap submission | Same day | 2026-07-18 | Measured |

**Still unmeasured, honestly:** the post-noindex-fix recovery curve (the re-measure was scheduled for 2–4 weeks out and postdates this record); any AI-answer citations (no battery was run for a local business this small in the window); the traffic cost of the 11 noindexed days and of the pre-migration challenge era (no analytics existed on the old site — and none was installed on the new one during the window, by explicit owner-gated decision); Business Profile impact (the listing remained unverified, and the Business Profile API remained at quota zero, at record end).

## What worked

- **Probing as bots before optimizing anything.** Two `curl` commands found the single biggest problem on day one of the SEO round — no tool subscription involved.
- **Adversarial review of the migration plan.** Twelve blockers caught on paper instead of in production, three of them damaging.
- **Hand-authored schema against the policy grain.** Ratings on a Service node, FAQ extracted from visible content, one `@id`-linked entity — everything validated and nothing eligible for a manual action.
- **The single-source-of-truth review pipeline.** After it landed, a number that had drifted three ways could only change in one place.
- **Evidence-gated local claims.** TIGER boundaries, city-limits geocoding, and verbatim reviews produced city pages that can survive both an algorithm update and a skeptical customer.

## What failed

- **The WAF challenge had already erased the domain** before the engagement began — the site looked fine to every human who checked it.
- **The domain field shipped 11 days of sitewide noindex.** Set during the cutover for a legitimate reason, on a platform where the proxy guarantees the check it drives can never pass. Caught only because GSC data was finally pulled programmatically.
- **Hardcoded social proof drifted immediately and repeatedly.** Every manual sweep was stale within days until the dynamic pipeline landed.
- **Generated content drifted into fabrication.** Testimonials that matched nobody and photos captioned as specific jobs — introduced by earlier automated build waves, caught late by a dedicated audit rather than early by review of the generators' output.
- **The citations program produced zero listings** — correct behavior at the automation boundary, but the NAP blockers (phone conflict, no publishable address) mean the off-site layer was still unbuilt at record end.
- **robots.txt hid the catalogue for weeks** — boilerplate `Disallow: /shop` was never diffed against what the shop actually contained.

## What we'd do differently

- **Put "fetch the live site as Googlebot and GPTBot, from outside" in the go-live checklist** — and re-run it after *every* domain or infrastructure change, not just the first launch.
- **Pull GSC data weekly from day one.** The noindex would have been a 3-day incident, not an 11-day one.
- **Never let a repeated number ship hardcoded.** The first "4.9 · 51 reviews" badge should have been a dynamic render.
- **Audit generated content the day it's generated.** The authenticity pass was a late-stage rescue; it should be a gate in the content pipeline.
- **Resolve canonical NAP before building anything that displays it** — the phone/address decision blocked the entire citations layer weeks after it could have been settled.

## The lessons, mapped

| Lesson | Chapter |
|---|---|
| A bot-challenge mode erases you from search and AI while the site looks fine | [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md) |
| Diagnose from the public surface: bot-UA fetches + `site:` + URL Inspection | [The 30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) |
| The CMS domain-field / Host-rewrite noindex trap | [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md) |
| Ratings and reviews go on a Service node, mirroring visible content only | [Reviews — real ones only](../local/reviews.md) |
| Generate FAQPage from the rendered page, never by hand | [FAQ schema from visible content](../local/faq-schema.md) |
| One sitewide `#business` entity node; per-page graphs reference it | [LocalBusiness schema graph](../local/local-business-schema.md) |
| City pages earn existence through verifiable evidence, not templates | [Service-area pages](../local/service-areas.md) |
| Fabricated content is a discoverability and legal risk, not a shortcut | [Authenticity audits](../local/authenticity.md) |
| Cutovers: pre-stage DNS, pre-issue certs, preserve machine hosts and email | [Domain migrations](../technical/domain-migration.md) |
| GSC is API-drivable end to end; verification tokens are per-account | [Google Search Console](../google/search-console.md) |
| Welcome AI crawlers explicitly; verify they can actually fetch | [AI crawlers and crawlability](../ai-search/ai-crawlers.md) |
| Directory/citation truth beats directory volume; NAP first | [Off-site signals](../ai-search/offsite-signals.md) |

## Chapters this case feeds

- [Local Business](../local/index.md) — the entire part is this case, generalized
- [Launch a local business](../playbooks/local-business.md) — this timeline as a runnable sequence
- [Google Business Profile](../google/business-profile.md) — the invisibility finding and the API-quota reality
- [Measurement and baselines](../foundations/measurement.md) — the baseline that caught the noindex
- [everjust-tenants](everjust-tenants.md) — the platform-side view of the same traps
- Source skills: [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo), [reverse-proxy-cms-indexing](https://github.com/ever-just/agentskills/tree/main/skills/reverse-proxy-cms-indexing), [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization) — plus the local-business AEO and authenticity-audit skills; full mapping in the [skill index](../appendix/skills-index.md)

---

*Record window: 2026-07-09 → 2026-07-30. Open items at close: Business Profile verification (owner), the citations human queue, analytics installation, and the post-fix recovery re-measure. Later developments will be appended below with dates; the record above stays as written.*
