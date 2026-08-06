# Launch a SaaS product

This is the sequence that took [customdomain.ai](../case-studies/customdomain-ai.md) from a brand-new, effectively invisible `.ai` domain to a Bing-verified, schema-validated, MCP-registered property in about three weeks of working sessions (July 2026). Run the phases in order — each one gates the next. Phases 0–1 fit in one afternoon; the whole program is 2–4 weeks for one operator, less if you parallelize the content work.

**Prerequisites:** a live site with valid TLS on the domain you intend to keep, control of DNS, and owner access to Google/Bing accounts. If you're mid-rebrand, do the [domain migration](../technical/domain-migration.md) first — every artifact below binds to the domain.

---

## Phase 0 — Record the baseline

**Goal:** measurement plumbing exists and the "before" numbers are written down. Nothing else ships until this is done, because a change made before the baseline is a change you can never evaluate.

- [ ] Verify a **Google Search Console** property — prefer a domain property (`sc-domain:`, covers all subdomains) via DNS TXT. If you'll automate, add a service account with `siteOwner` permission; the whole API surface (performance queries, sitemap submission, URL Inspection) works headlessly. → [Search Console](../google/search-console.md)
- [ ] Check whether your sitemap has **ever been submitted** — customdomain.ai's 101-URL sitemap never had been; submitting it was a free win found on day one.
- [ ] Record the 90-day baseline: clicks, impressions, average position, and the full query list with positions. Save it to your log file.
- [ ] Verify **Bing Webmaster Tools** (import from GSC takes minutes). Bing feeds ChatGPT search — BWT's reports are the only place you can *observe* that pipe, and running without it was graded the #1 leverage-per-effort gap in the source audit. → [Bing Webmaster Tools](../bing/bing-webmaster-tools.md)
- [ ] Run the [30-minute AI visibility audit](ai-visibility-30min.md) and file the scorecard as your AI-surface baseline.

**Effort:** Hours (1–3, mostly account/DNS plumbing).
**Verification:** you can open a file containing dated numbers for clicks / impressions / position and an AI-visibility scorecard. GSC and BWT both show the property verified.
**Deep dives:** [Measurement and baselines](../foundations/measurement.md)

!!! note "What a real baseline looks like"
    customdomain.ai, 90 days to 2026-07-15: **5 clicks, 159 impressions, average position 42.5** — indexed but invisible, with impressions on the right intents at unrankable positions. That one line made every later claim of progress checkable.

---

## Phase 1 — Prove you're crawlable

**Goal:** Googlebot, Bingbot, and the AI crawler roster can all fetch your pages, and every host serves the truth.

- [ ] Curl the homepage, docs, and one key page as Googlebot, GPTBot, OAI-SearchBot, ClaudeBot, PerplexityBot, and Bingbot — expect 200s, no challenge pages. Commands and interpretation: [30-minute audit, step 1](ai-visibility-30min.md#minutes-06-fetch-the-site-the-way-bots-do). → [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md)
- [ ] Fetch the **public** `robots.txt` and read it end to end. Add explicit Allow groups for the AI crawler roster. → [AI crawlers](../ai-search/ai-crawlers.md), [Sitemaps and robots](../google/sitemaps-and-robots.md)
- [ ] If you're behind a reverse proxy or multi-tenant CMS: check that canonicals, the robots `Sitemap:` line, and sitemap `<loc>` entries all show the **public** host, and that the internal host isn't a crawlable duplicate (noindex it at the proxy, don't redirect it). → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)
- [ ] Confirm key content is server-rendered — view-source shows your copy, not a JS shell.

**Effort:** Minutes to check; Hours if a WAF or proxy fix is needed.
**Verification:** all bot-UA curls return 200 with real HTML; robots.txt read and correct on the public URL; `site:yourdomain` returns pages on Google and Bing (a young domain may legitimately show few — see the timeline note in Gotchas).
**Deep dives:** [How AI finds and cites](../foundations/ai-retrieval.md)

---

## Phase 2 — Ship the schema graph

**Goal:** one coherent JSON-LD `@graph` that tells machines what you are, what you do, and what it costs — the dual **SoftwareApplication + Service** model.

- [ ] Inventory what's already live: `curl -s https://yourdomain/ | grep -o '"@type": *"[A-Za-z]*"' | sort | uniq -c`. If your CMS auto-emits an Organization/WebSite block, keep it and **share its `@id`** — never run two competing Organization nodes.
- [ ] Model the product as **two cross-linked nodes**: `SoftwareApplication` (+`WebApplication`) with `applicationCategory` and a rich `featureList` — the only node Google renders a software rich result for — and a `Service` node carrying `hasOfferCatalog`, because the catalog property is **not valid on SoftwareApplication**. Cross-link everything by `@id`. → [Structured data](../google/structured-data.md), [Rich results](../google/rich-results.md)
- [ ] Pricing: one `Offer` per visible tier; recurring prices as `UnitPriceSpecification` (price + monthly billing terms); **custom-priced tiers get no price node at all** (a `"0"` reads as "free"). Markup prices must equal on-page prices.
- [ ] Skip `aggregateRating` until real reviews are visible on the page — fabricating it is a spam-policy violation.
- [ ] Optional but high-leverage for a dev tool: `potentialAction`/`EntryPoint` nodes on the Organization pointing at your public API, OpenAPI document, and MCP endpoint — machine-readable signposts from the human surface to the agent surface.
- [ ] Build the block as code and validate offline (parse the JSON, count types, check each Offer's shape) **before** touching the site.

**Effort:** Day+ (research + modeling + deployment). The worked example went from design to validated-live in ~30 hours — half of which was fighting the CMS deployment, not the schema (see Gotchas).
**Verification:** three layers, all required — (1) the JSON parses and the offline census matches your design; (2) the **live rendered HTML** contains the block (fetch it; a saved CMS record is not verification); (3) Google's Rich Results Test shows your page eligible with zero errors, and validator.schema.org parses the full graph (RRT only displays Google's visual rich-result types — Service/OfferCatalog absent from its report is expected, not failure).
**Deep dives:** [Rich results](../google/rich-results.md), [Templates](../appendix/templates.md)

---

## Phase 3 — Docs site and llms.txt

**Goal:** a crawlable, server-rendered docs surface, plus an `llms.txt` sized to what it actually does.

- [ ] Ship docs on a crawlable, SSR host (subdomain is fine under an `sc-domain` property). Per-operation API reference pages give agents and answer engines something specific to retrieve.
- [ ] Add `/llms.txt` as a **curated index** — H1, a one-paragraph factual answer block, key-facts bullets, described links — not an auto-generated page dump. If you ship `llms-full.txt`, make sure it emits clean content: the source program found its own emitting raw React JSX for all 47 API pages.
- [ ] Include a **"For AI agents"** section naming your MCP endpoint, registry name, and runbook link — an LLM reading the human surface otherwise cannot find the agent surface.
- [ ] Calibrate effort honestly: research at the time (Ahrefs, 137k domains, 2026) found **~97% of llms.txt files get zero bot requests**, and Google says it doesn't use the file. Its real consumers are coding agents. Ship it cheap; spend the saved time on crawlability and Bing. → [llms.txt — the reality check](../ai-search/llms-txt.md)

**Effort:** Hours (assuming docs exist; standing up a docs site is its own project).
**Verification:** `curl https://yourdomain/llms.txt` returns the curated file; docs pages return full HTML to a bot-UA curl.
**Deep dives:** [llms.txt](../ai-search/llms-txt.md), [GitHub as a discovery surface](../agents/github-as-discovery.md)

---

## Phase 4 — Keyword reality check

**Goal:** you know which queries are winnable, and you've caught the brand-name trap before writing content.

- [ ] **Tokenization check:** search your exact brand name. A dotted or generic-word brand ("customdomain.ai") can tokenize into generic words and return **zero results for you** — the SERP goes to whoever owns the generic phrase. If that's you, the fix is an entity program (Organization schema + `sameAs`, consistent profiles, always writing the brand one-token externally), expected to take 3–6 months. → [Keyword strategy](../google/keyword-strategy.md)
- [ ] **Head-term realism:** run live SERP recon on your dream term. If page one is DR90+ platforms and navigational intent, it's a 24–36-month project and mostly not worth it. Pick the *operational* head term instead ("custom domains for SaaS", not "custom domain").
- [ ] **Find the vacant niches:** SERPs whose #1 is a thin Medium post, a dead page, or a Quora answer are winnable in weeks-to-months. Category terms nobody claimed ("MCP for domains") are the land grab — first-mover windows close.
- [ ] **Mine striking distance:** pull your GSC query list; anything ranked #30–70 is proven demand where Google already considers you relevant — the fastest terms to move.
- [ ] Score candidates (demand × relevance × business value ÷ difficulty), map each to a page type, and write the 90-day plan.

**Effort:** Hours to Day+ (the SERP recon is the slow, valuable part).
**Verification:** a written list of term families with verdicts (winnable / vacant / park), each mapped to a page that will target it.
**Deep dives:** [Keyword and SERP strategy](../google/keyword-strategy.md), [GEO fundamentals](../ai-search/geo-fundamentals.md)

---

## Phase 5 — Build the content cluster

**Goal:** a pillar + glossary + guides cluster that answers intent queries in liftable, citable form.

- [ ] One **pillar page** on your operational head term: answer-first (a 40–60-word direct answer up top), question-shaped H2s each answered in its first sentence, a comparison table (research-reported: tables get extracted ~81% vs ~23% for prose), FAQ block with FAQPage markup.
- [ ] A **glossary** of the category's terms (DefinedTerm + FAQPage per entry, CollectionPage index) — definitions written to be quoted verbatim. If the category name is contested, publish the one-sentence definition you want AIs to lift.
- [ ] Two-plus **developer guides** (TechArticle + FAQPage) on the problems your ICP literally types.
- [ ] Wire **hub-and-spoke internal links** and a footer "Resources" column so the cluster gets site-wide link equity.
- [ ] On every publish: confirm the page reaches the sitemap (mind CMS sitemap caching), and ping IndexNow. → [IndexNow](../bing/indexnow.md)

**Effort:** Day+ per wave for a solo operator — realistically weeks for a full cluster. (The source program shipped a pillar + 4 glossary entries + 3 guides in a day, and later 15 pages in a day, using parallel agent fleets; budget accordingly if you're one human.)
**Verification:** each page live, in the sitemap, emitting its schema types, and answering its target query in the first 200 words. Re-run two battery queries from your Phase 0 scorecard that the new pages should influence — expect movement over weeks, not days.
**Deep dives:** [Content that gets cited](../ai-search/content-that-gets-cited.md), [Off-site signals](../ai-search/offsite-signals.md)

---

## Phase 6 — MCP registry and OAuth discovery *(if you have an API or MCP server)*

**Goal:** an agent that hears about you can find the endpoint, authenticate, and finish a job. Sequencing rule from the field: **endpoint first, discovery records second** — registry entries pointing at a 404 are hollow.

- [ ] Confirm the MCP endpoint is live and in git — the source audit found the entire discovery layer (well-knowns, manifest, registry entry) live in prod but absent from the repo, meaning a rebuild would silently erase it. Version `server.json` in-repo and add a CI check that served == advertised.
- [ ] Publish to the **official MCP Registry** (reverse-DNS name, DNS TXT domain verification). Community directories largely mirror it — publish once upstream, then claim your listings. → [The MCP Registry](../agents/mcp-registry.md)
- [ ] Test the **OAuth discovery chain** end to end: the 401 must carry `WWW-Authenticate … resource_metadata=…` → `/.well-known/oauth-protected-resource` → `/.well-known/oauth-authorization-server` → live token endpoint. A broken chain is the #1 silent connection failure. → [OAuth discovery chain](../agents/oauth-discovery.md)
- [ ] Add **tool annotations** (title, read-only/destructive hints) to every tool — connector directories hard-require them — and rewrite descriptions as outcome + when-to-use in your ICP's phrasing. → [Tool descriptions that rank](../agents/tool-descriptions.md)
- [ ] Optionally add the capability manifest and DNS-AID records, and verify the whole chain resolves (a dangling index record is a real observed failure). → [Manifests and DNS-AID](../agents/manifests-and-dns.md)

**Effort:** Day+ (registry + chain testing in hours; annotation/description quality is the real work).
**Verification:** registry entry shows active; an MCP client with no prior knowledge completes the connect flow; every well-known URL returns valid JSON over a plain curl.
**Deep dives:** [AI Agents overview](../agents/index.md)

---

## Phase 7 — The GitHub org funnel

**Goal:** your GitHub org ranks for your category's queries and routes readers (and crawlers — Bing has a privileged GitHub crawl that feeds ChatGPT) to the product.

- [ ] Know the index: GitHub repo search indexes **only name, description, and topics** — not README text. Put your target phrases there; topics are exact-match and category topics are often unclaimed.
- [ ] Org profile README as the front door: what the product does, repo map, deep links to docs and the registry listing.
- [ ] A public **docs repo** as source of truth (content-only, secret-scanned), plus audience-targeted starter repos with deep READMEs and an `AGENTS.md` (the agent-readable standard — 20+ tools auto-read it; an in-repo llms.txt has almost no consumers).
- [ ] Create or PR into the category's **awesome-list**, honestly listing alternatives.
- [ ] Remember only `/blob/` pages are crawlable — link to files, not `/tree/` or `/raw/`.

**Effort:** Day+ for the initial buildout; Minutes per repo thereafter.
**Verification:** searching GitHub for your 3–5 target phrases returns your repos on page one (many of these SERPs are near-empty — winnable at zero stars).
**Deep dives:** [GitHub as a discovery surface](../agents/github-as-discovery.md)

---

## Phase 8 — The Ask-AI widget

**Goal:** a footer/docs row of deep links that open ChatGPT, Perplexity, Claude, Google AI Mode, and Grok pre-loaded with a grounded prompt about your product.

- [ ] Use only verified schemes (as of 2026-07: `chatgpt.com/?q=` auto-submits; `perplexity.ai/search?q=` auto-runs; `google.com/search?udm=50&q=`; `claude.ai/new?q=` prefills; `grok.com/?q=`). Skip Gemini and Copilot — their `?q=` is ignored or broken.
- [ ] Make the prompt **self-contained**: bake 4–6 accurate product facts into it. A real click-test showed ChatGPT *searches* (via Bing) rather than fetching a URL from the prompt — an unindexed site yields nothing, and telling it to "read our llms.txt" returns articles about the llms.txt format.
- [ ] Schemes are undocumented and break — put re-verification on the [quarterly cadence](operating-cadence.md#quarterly-half-a-day).

**Effort:** Hours.
**Verification:** click every link in a clean browser session; each opens the right provider with the prompt intact and the answer is factually correct.
**Deep dives:** [The Ask-AI widget](../ai-search/ask-ai-widget.md)

---

## Phase 9 — Turn on the measurement loop

**Goal:** the system keeps itself fresh and you keep score.

- [ ] Automate freshness: an IndexNow sweep on a **6-hourly cron** (reads the live sitemap, submits new/changed URLs, weekly full re-submit) and — if your CMS caches its sitemap — a sitemap-cache-clearing job so new pages appear within hours. → [IndexNow](../bing/indexnow.md)
- [ ] Accept the Google reality: there is **no legitimate auto-submit** for a marketing site. The sitemap ping endpoint died in 2023; the Indexing API covers JobPosting/BroadcastEvent only, and off-label use risks revocation. Google freshness = a correct sitemap + the one-time GSC submission; it re-crawls forever.
- [ ] Put the [operating cadence](operating-cadence.md) on your calendar: weekly GSC/BWT glance, monthly striking-distance mining, quarterly full re-audit.

**Effort:** Hours to set up; Ongoing thereafter.
**Verification:** the cron logs show submissions accepted (HTTP 200/202); a page published today appears in the sitemap within a few hours; the first weekly review actually happens.

---

## Gotchas

The failures this sequence was built around — hit in production, in this order of pain:

1. **Verification files served by the CMS flap during deploys.** Bing's verification fetch 404'd exactly when the app restarted. Serve `BingSiteAuth.xml` and the IndexNow key from the edge (an exact-match nginx `location`), with the meta-tag method as redundancy. → [Bing Webmaster Tools](../bing/bing-webmaster-tools.md)
2. **"Verified in the CMS" ≠ rendered.** The homepage schema sat correctly in a stored view that `/` never actually rendered (the site's homepage setting routed `/` to a different page), surviving cache-flush theories and a container restart's worth of debugging. Always prove which template serves a route (distinctive-string test) and verify the live HTML. → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)
3. **Two JSON-LD emitters, one entity.** A platform-emitted Organization block plus your hand-authored one = duplicate orgs unless they share an `@id`.
4. **`hasOfferCatalog` on SoftwareApplication is invalid.** This single vocabulary constraint forces the dual-node model; validators won't always save you.
5. **The brand name itself can be unsearchable.** Test tokenization *before* investing in branded-SERP dreams (Phase 4).
6. **Bing unverified = running blind.** IndexNow 202s prove submission, not reception; only BWT shows you Bing coverage and Copilot/AI performance.
7. **Young-domain patience.** First pages index in ~2–4 weeks, fuller coverage ~2–3 months. A quiet week one is not a failure signal; re-measure on the cadence, not daily.
8. **Discovery surface not in git.** If the well-knowns, manifest, and registry file exist only in prod config, a rebuild deletes your agent presence silently.

## What happened when we ran it

On record for customdomain.ai (July 2026): the only Bing-verified property in its portfolio, with automated IndexNow + sitemap-freshness loops; a live dual-node schema graph validated by Google's Rich Results Test ("3 valid items", zero errors) within ~30 hours of design; an MCP server listed in the official registry with a textbook-complete OAuth discovery chain; a 117-URL content footprint up from 93. Ranking and AI-citation movement from the young-domain baseline (5 clicks / 159 impressions / position 42.5) was **unmeasured within the source window** — the measurement loop exists precisely to answer it on schedule.

## Related

- [customdomain.ai case study](../case-studies/customdomain-ai.md) — the full narrative with dates
- [Launch a local business](local-business.md) — the sibling sequence
- [The 30-minute AI visibility audit](ai-visibility-30min.md) — Phase 0/1 as a standalone diagnostic
- [The operating cadence](operating-cadence.md) — Phase 9, expanded
- [The master checklist](../start/master-checklist.md) — every item here, flattened
