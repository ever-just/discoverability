# Launch a local business

This is the sequence that rebuilt [Heads Up Outdoor Services](../case-studies/headsup.md) (July 2026) — a real lawn-care company whose site turned out to be **completely invisible to search and AI** before any optimization mattered: first behind a WAF bot-challenge that 429'd Googlebot, GPTBot, and PerplexityBot alike (`site:` showed zero pages; the company's Facebook page outranked its own domain), then, after migration, silently `noindex`-ed on every page by a CMS trap for 11 days. The lesson is the ordering: **check the gate first**. Everything else in local SEO — GBP, schema, reviews, city pages — builds on a site crawlers can actually read.

**Prerequisites:** a live site on the business's real domain (or a [migration plan](../technical/domain-migration.md) to get there), owner access to the Google account, and the discipline to publish only claims you can prove — this playbook treats honesty as an SEO architecture, not a virtue.

---

## Phase 0 — Crawlability and baseline (the WAF check first)

**Goal:** you have proven crawlers can read the site, and the "before" numbers are on file.

- [ ] **The invisibility check, before anything else:** curl the homepage, `robots.txt`, and `sitemap.xml` as a browser, Googlebot, GPTBot, and PerplexityBot (commands: [30-minute audit](ai-visibility-30min.md#minutes-06-fetch-the-site-the-way-bots-do)). A 403/429/challenge response — the real case returned HTTP 429 with a `x-vercel-mitigated: challenge` header on *every* URL including robots.txt — means a security toggle has removed you from search and AI simultaneously. Fix is configuration, not content. → [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md)
- [ ] Run `site:yourdomain.com` on Google and Bing. Zero results on an established domain = crisis confirmed.
- [ ] Verify **Google Search Console** (domain property; DNS TXT; each Google account needs its own verification token, so read existing TXT records first and never clobber SPF). Record the 90-day baseline: clicks, impressions, position, and *which queries* — the case baseline was 24 clicks / 640 impressions / position 14.5, essentially all brand-name searches, which told us commercial terms were nowhere. → [Search Console](../google/search-console.md)
- [ ] Run **URL Inspection** on 3–5 key pages. This is what caught the second catastrophe: every page `Excluded by 'noindex' tag` — Google crawling daily and discarding — caused by a CMS domain-field trap behind the reverse proxy. → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)
- [ ] Submit the sitemap; set up **Bing Webmaster Tools** while you're at it. → [Bing Webmaster Tools](../bing/bing-webmaster-tools.md)

**Effort:** Minutes for the checks; Hours if GSC/DNS setup is from scratch; a WAF or noindex fix is usually a config change once found.
**Verification:** all bot-UA curls return 200 with real HTML; URL Inspection shows pages indexable; baseline numbers are written down and dated.

---

## Phase 1 — Google Business Profile

**Goal:** the single highest-impact local surface is claimed, complete, and *actually findable*.

- [ ] **Test that the listing resolves:** search the business name (and name + city) in Google Maps. In the real case, Places text search returned zero results for the business across name/address/phone queries while competitors resolved instantly — a live business, invisible in local search. If that's you, the profile is unverified/unpublished; fixing it outranks everything else in this playbook.
- [ ] Claim and verify the profile; complete every field: categories, services (with real prices where you publish them), hours, description, service areas, photos, first post.
- [ ] Know the API reality: the Business Profile API is OAuth-only and ships at **quota 0 pending Google's access approval** (a 429 with an empty quota bucket is the pending state, not rate limiting — don't retry). Apply early if you'll automate; do today's work in the signed-in browser.
- [ ] Wire the review loop: the `writereview` deep link on your site, plus a post-job review-request email. Reviews are the local ranking currency, and research at the time put the AI-recommendation confidence bar around a 4.3-star average.

**Effort:** Hours (plus Google's verification lead time, which can be days).
**Verification:** the business resolves in a Maps/Places search by name; the profile shows complete; the website link points at the real domain.
**Deep dives:** [Google Business Profile](../google/business-profile.md), [Reviews](../local/reviews.md)

---

## Phase 2 — The sitewide LocalBusiness graph

**Goal:** every page emits one authoritative `LocalBusiness` node machines can resolve as a single confident entity.

- [ ] Inject **one** full `LocalBusiness` node (use the specific subtype, e.g. `LandscapingBusiness`) with a stable `@id` (`#business`) sitewide in `<head>`: name, phone, email, address, geo, priceRange, foundingDate, `areaServed` (one `City` object per real service city), opening hours, `sameAs` (Facebook, Instagram, Yelp, BBB), and `hasMap` pointing at the Maps listing — this ties the site entity to the Business Profile.
- [ ] Point per-page `Service` nodes' `provider` at `{"@id": "#business"}` — the case found Service blocks referencing a `#business` that didn't exist anywhere (a dangling graph is worse than no graph).
- [ ] **No `aggregateRating` on this node** — that's Phase 4's policy trap.
- [ ] Consolidate duplicates: the audit found two conflicting Organization blocks per page (different logos, no `@id`). One entity, one `@id`, everywhere.

**Effort:** Hours (half a day with CMS wrestling).
**Verification:** fetch the **live rendered HTML** of three page types and parse the JSON-LD — the case's node was "deployed" but shadowed by a deactivated template fork for weeks; only the rendered page tells the truth. Then validator.schema.org, then Rich Results Test.
**Deep dives:** [LocalBusiness schema graph](../local/local-business-schema.md), [Entities and trust](../foundations/entities-and-trust.md), [Templates](../appendix/templates.md)

---

## Phase 3 — Per-page Service, FAQ, and Breadcrumb schema

**Goal:** every service and city page carries 3+ cross-linked schema types — the strongest on-site correlate with AI citation in the research behind this program (rich schema on ~61% of ChatGPT-cited pages vs ~25% baseline; research-reported).

- [ ] Each service page: `@graph` of `Service` (serviceType, `provider → #business`, areaServed) + `FAQPage` + `BreadcrumbList`.
- [ ] Build FAQPage **deterministically from the page's visible Q&A** — extract the rendered accordion/heading markup rather than hand-authoring, so schema can never drift from what's on the page (a Google requirement and an anti-drift device). → [FAQ schema from visible content](../local/faq-schema.md)
- [ ] City pages: same graph, with the `City` in `areaServed` carrying a Wikipedia `sameAs` for entity disambiguation.
- [ ] Note the date-stamped reality: Google dropped FAQ rich results (May 2026, as cited in the source research), but FAQ-structured content still maps to how people prompt AIs and Bing still ingests it — keep visible FAQs + markup, skip the SERP-stars expectation.

**Effort:** Hours-to-Day+ depending on page count (the case covered 15 service pages in one evening with an extractor script).
**Verification:** every page class emits its full graph in rendered HTML; Rich Results Test passes breadcrumbs; no schema-vs-visible mismatch (the case's quote-tool FAQ markup once told Google prices "didn't exist" after a price edit — that's the failure mode).

---

## Phase 4 — Real reviews, synced live

**Goal:** review markup that is policy-safe, and displayed counts that can't drift from reality.

- [ ] **The policy rule (Google, restated December 2025):** self-serving `aggregateRating`/`review` on LocalBusiness/Organization is ineligible and risks a manual action. Stars go on a **Service** node, backed by reviews *visible on that page* — real author names, verbatim bodies. Never fabricate; omit until real reviews are on-page. → [Reviews — real ones only](../local/reviews.md)
- [ ] Sync from the source: Places API returns rating + count but **caps at 5 reviews per pull** on any key; the only route to all reviews is the approval-gated Business Profile API. Store and accumulate what you can pull; display verbatim.
- [ ] Kill hardcoded social proof: the case had "4.9 · 51 reviews" hardcoded in ~35–90 places and caught it drifting (47 vs 48 vs 51 across pages) within days. One stored value (a config parameter), rendered dynamically everywhere — badges, aria-labels, meta descriptions, JSON-LD `reviewCount` — patched by a scheduled job.
- [ ] Run the sync **outside** your CMS if its automation sandbox blocks HTTP (the case's did); a 6-hourly-to-daily cron is plenty.
- [ ] Sweep the invisible surfaces: SEO meta titles/descriptions are separate CMS fields that greps of page content miss — the one spot that silently drifted in production.

**Effort:** Day+ for the full pipeline; Hours for a manual-update-with-checklist version (fine at small scale).
**Verification:** displayed rating/count matches the live Google listing today; a week later, still matches; the reviews page emits Review nodes only for reviews a visitor can read on it.

---

## Phase 5 — Service-area truthing

**Goal:** service-area pages that survive both Google's doorway-page policies and an honest audit — every locality claim traceable to evidence.

- [ ] **Kill the doorway pages first.** The case 301'd and de-indexed 22 thin street/neighborhood pages (three of them advertising service on land the business couldn't serve). City-swap template pages are a penalty class, not a strategy.
- [ ] A city page exists only where the business has **real presence** — pull leads/customers per city from your CRM and weight page depth by demand (the case: 64 leads in the home city, 0–2 in edge cities; the 0-lead city got a thin page, not a shrine).
- [ ] Photos carry a city claim only with **EXIF-GPS proof that survives a city-limits geocode check** — postal city ≠ municipal boundary; the case caught a landmark photo whose camera position sat in the neighboring city. Keep two registers, never blended: WORK (your jobs, provable) vs PLACE (credited town photos, explicitly labeled as the town).
- [ ] City-attributed testimonials only from reviews **already published with that city**. The case found 7 testimonials matching no CRM record or published review next to a "verified reviews" claim — an FTC exposure, replaced with verbatim real reviews.
- [ ] Maps and distances without the ToS traps: public-domain Census TIGER boundary SVGs beat embedded map products; publish straight-line miles, never Google-measured drive times (those are licensed "Maps Content").

**Effort:** Day+ (the honest version is real work — that's why it ranks).
**Verification:** every claim on every city page traces to CRM data, GPS+geocode, a published review, or a license record; anything unverifiable is labeled or removed; former doorway URLs 301 to their city page.
**Deep dives:** [Service-area pages](../local/service-areas.md)

---

## Phase 6 — The authenticity pass

**Goal:** nothing on the site can be falsified by a skeptical reader, a competitor, a regulator — or an AI summarizing you.

- [ ] Fetch-and-compare every image against its caption; de-caption stock and AI images presented as your work (the audit's severest class: an AI-generated hero declared in `ImageObject` markup as a photo of a specific job).
- [ ] Hunt invented pull-quotes, unattributable testimonials, and fabricated statistics; replace with verbatim real material or labeled generic examples.
- [ ] Check badge claims against the issuer: the case's site displayed a "BBB Accredited" badge while BBB's own profile said not accredited — rewritten to the true "A+ rated" claim.
- [ ] Reconcile numbers that disagree across pages (prices, counts, "150+ properties" vs "70 clients") — inconsistency is both a trust and a schema problem.

**Effort:** Hours-to-Day+ depending on how much history the site carries.
**Verification:** an adversarial re-read (ideally by someone else) finds zero claims that can't be traced to evidence.
**Deep dives:** [Authenticity audits](../local/authenticity.md)

---

## Phase 7 — The citations program

**Goal:** identical NAP (name, address, phone) on the ~20 directories that matter — research at the time put consistent businesses at ~3x the likelihood of AI local recommendation, because assistants pull contact facts from a narrow trusted set (Yelp, BBB, aggregators). A wrong number there means the AI hallucinates the wrong contact.

- [ ] **Decide the canonical NAP first** — one phone, one address form, one description set. Correcting after 50 profiles exist means editing 50 profiles. (The case hit a real blocker here: a personal cell on old collateral vs the tracked business line — resolve before creating anything.)
- [ ] **Claim before creating:** search each directory by name and phone for existing/duplicate listings first.
- [ ] Triage the platform list ruthlessly. Of ~50 platforms researched for the case, **20 were worth doing**; 12 were executable but worthless (sequenced last); 17 were excluded (paid-only, dead, or demanding a street address the owner won't publish). An empty profile on a big domain does ~nothing; a half-built one with mismatched NAP actively hurts.
- [ ] Split the work honestly: research, form-filling, and verification are delegable; account creation, passwords, and CAPTCHAs are the owner's. Log every listing (platform, URL, NAP used, status) — this log is your monthly drift check.
- [ ] Platform quirks are volatile (one 2026 example: Apple's business program changed enrollment requirements that spring, obsoleting every older guide) — verify each platform's current flow, don't trust posts.

**Effort:** Ongoing over 2–4 weeks in Hours-sized sessions; front-load GBP, Bing Places (which feeds Copilot/ChatGPT grounding), Apple, Yelp, BBB, Facebook, Nextdoor.
**Verification:** the citation log shows each listing live with byte-identical NAP; a monthly spot-check of the top five shows no drift.
**Deep dives:** [Off-site signals](../ai-search/offsite-signals.md), [Entities and trust](../foundations/entities-and-trust.md)

---

## Phase 8 — The AI crawler allow-list

**Goal:** every AI assistant that could recommend you can read you, and you've verified it end to end.

- [ ] Add explicit `Allow` groups to robots.txt for the roster — GPTBot, OAI-SearchBot, ChatGPT-User, ClaudeBot, Claude-SearchBot, PerplexityBot, Google-Extended (roster as of 2026; keep it current via the [crawler registry](../appendix/crawler-registry.md)). For a local service business there's rarely an IP-protection reason to block training bots either — being in the training data is free brand distribution.
- [ ] Keep the plumbing paths disallowed (portal, cart, session URLs) — but **diff robots.txt against your sitemap** first: the case's boilerplate `Disallow: /shop` was hiding 14 real, sellable service pages.
- [ ] Re-verify from outside after every robots change, on the **public** host: the case found nginx serving a stale robots.txt on the public domain while the internal host had the new one.

**Effort:** Minutes (plus whatever cache-purge plumbing your stack needs).
**Verification:** the public `robots.txt` shows the allow-list; bot-UA curls still return 200; no sitemap URL is robots-blocked.
**Deep dives:** [AI crawlers and crawlability](../ai-search/ai-crawlers.md), [Sitemaps and robots.txt](../google/sitemaps-and-robots.md)

---

## Phase 9 — Hand off to the cadence

- [ ] Automate what machines should own: the review patcher (Phase 4) and, if your platform supports it, IndexNow pings on publish. → [IndexNow](../bing/indexnow.md)
- [ ] Calendar the human rhythm: weekly GSC/BWT glance, monthly review-sync + citation-drift check, quarterly re-audit. → [The operating cadence](operating-cadence.md)
- [ ] Re-measure against the Phase 0 baseline in 2–4 weeks — indexing recovers on Google's schedule, not yours.

---

## Gotchas

The production failures this ordering exists to catch:

1. **A security toggle can erase you.** The WAF challenge blocked *everything* — including robots.txt — while humans saw a working site. Detect by fetching as bots + `site:`; never assume.
2. **The CMS domain-field noindex trap.** On a Host-rewriting proxy stack, setting the CMS's "domain" field noindexed every page and emitted `Disallow: /` — and a parallel session *re-set* the field to fix a cosmetic og:url issue, re-noindexing the site within seconds. Know your platform's trap before touching domain settings. → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)
3. **Layered caches eat directive changes.** The sitemap was a ~12-hour CMS-cached attachment; robots.txt was separately cached by nginx on the public host where `no-store` didn't help. Verify every robots/sitemap change on the public URL, and know your cache-flush verbs.
4. **The sitemap and robots must agree.** The case's sitemap advertised URLs robots.txt forbade — GSC flags it and it burns crawl budget. Diff them whenever either changes.
5. **Review-count drift is inevitable if hardcoded.** 47 vs 48 vs 51 across surfaces within days — including inside aria-labels, SVG path data (blanket find-replace corrupts it), and meta-description fields greps don't see.
6. **"API enabled" ≠ usable.** Business Profile API at quota 0 means pending approval — don't retry, don't build on it; use the browser lane meanwhile.
7. **GBP can be live yet invisible** (unverified/suspended profiles return nothing in Places search). Test findability, not just existence.
8. **Doorway pages and fake local proof backfire** — with Google (Aug/Dec 2025 spam updates targeting scaled thin local pages, as cited in the source research), with the FTC, and with AI engines that quote your pages verbatim.

## What happened when we ran it

On record for headsupoutdoorservices.com (July 2026): the domain moved off the challenge wall onto the rebuilt site; robots, sitemap, and canonicals correct on the public host; GSC verified with the sitemap fetched same-day; the full entity graph (LocalBusiness + Service + FAQPage + Breadcrumb + Organization + WebSite) rendering on every page class; review counts made self-updating; 22 doorway pages retired; city pages rebuilt on provable claims; citations program triaged and logged. The sitewide-noindex trap was found and fixed on 2026-07-29 with the baseline recorded the same day — **the post-fix ranking recovery postdates the source material and is unmeasured here**. Open items at the time: GBP verification (owner-gated), Business Profile API quota, and the citation build-out's account-creation queue.

## Related

- [Heads Up case study](../case-studies/headsup.md) — the full narrative with dates
- [Local Business overview](../local/index.md) — the five local chapters this playbook sequences
- [Google Business Profile](../google/business-profile.md) · [Reviews](../local/reviews.md) · [Service-area pages](../local/service-areas.md) · [Authenticity audits](../local/authenticity.md)
- [The 30-minute AI visibility audit](ai-visibility-30min.md) — Phase 0 as a standalone diagnostic
- [The operating cadence](operating-cadence.md) — the rhythm this hands off to
