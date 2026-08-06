# Keyword and SERP strategy

**Most keyword strategy fails for one of two reasons: the brand name itself is unsearchable, or the team is chasing a head term it cannot win for three years.** This chapter covers both. It opens with the tokenization crisis — a real product whose *branded* search returned zero results for it — then gives the winnability triage we used to redirect that program: how to read a live SERP for weakness, how to score a vacant niche, how to turn the result into a 90-day battle plan, and how to allocate content across money pages, cost guides, and seasonal work. Every verdict below came from looking at live search results, not from a keyword tool's difficulty score.

## The brand-name tokenization crisis

**Searching `customdomain.ai` on Google returned zero results for customdomain.ai** (verified 2026-07-15). Not a low ranking — no result at all on page one. Google tokenizes the dotted name into the generic words that compose it, **"custom domain" + "ai"**, and serves what those words mean to everyone else: domain registrars, DNS help pages, and AI website-builder marketing.

This is the branded half of discovery — the half every guide calls "easy" — breaking on day one.

### Why it happens

Search engines don't see a brand; they see tokens. A brand made of common words separated by a dot is indistinguishable from the phrase it's made of, and the phrase already has owners with two decades of authority. Three compounding effects:

- **The generic phrase outweighs you.** "Custom domain" is a category noun with enormous existing intent. You are a new, low-authority site claiming a word the whole internet already uses.
- **The exact-match-domain bonus is long dead** (Google retired it in 2012). Owning the keyword as your domain buys you nothing on the ranking side while costing you your own branded SERP.
- **Written form makes it worse.** Every time the brand is written as two spaced words in a press mention or directory listing, the mention reinforces the generic phrase instead of your entity.

The same trap catches any name assembled from category words — dotted TLD or not. It is a *naming* decision with permanent discovery consequences.

### The test to run before you commit to a name

Ten minutes, before the domain purchase:

1. Search the exact name, logged out. Does anything relevant come back for a same-named entity, or does the SERP belong to the generic phrase?
2. Search the name as two words. If the results are identical, the engine is already treating them as the same string.
3. Ask ChatGPT and Perplexity "what is `<name>`?" and see whether they describe a concept rather than a company.
4. Check whether an established company already owns the phrase in your category.

If the branded SERP is already someone else's, you're buying a multi-quarter entity project along with the domain.

### If you're already stuck with the name

You do not rebrand over this — you run a **brand-entity program**, which is the same work as [entity building](../foundations/entities-and-trust.md) with unusual urgency. What we shipped first on customdomain.ai:

- **`Organization` schema with `alternateName`** and a real `sameAs` set — GitHub, LinkedIn, Crunchbase, directories — with **identical** name, logo, and description everywhere. One lone `sameAs` is not a corroboration graph.
- **A Wikidata item**, because entity databases are what assistants consult when they cannot verify an unfamiliar brand.
- **Always write the brand as one token externally** — `CustomDomain.ai`, never `custom domain ai`. Every spaced mention teaches the engine the wrong lesson. Put this in the brand guidelines and enforce it in press, directories, and profiles.
- **A homepage H1 that states the brand and the category together**, so the page can be matched to the name rather than only to the phrase.

**Expected timeline: 3–6 months to own your own branded SERP** — that is an informed estimate from the program, **not a measured outcome**; the recovery postdates the source material.

**How you know it worked:** the exact-name query returns your site first, ideally with sitelinks; a knowledge panel appears; and Search Console's Performance report shows the brand term climbing toward position 1 with rising impressions.

## Head-term realism

The head term you want is usually a 24–36 month project, and often not worth it even then. Read the *live SERP* rather than a difficulty score — the page types tell you what Google thinks the query means, and whether that's a job you can do better.

The verdict we recorded for the generic head term "custom domain" (2026-07): **not winnable in 24–36 months and mostly not worth winning.** The SERP was consumer/registrar intent and platform-navigational intent (people looking for how to set a custom domain on a specific platform), held by DR90+ properties. Even a #1 would have delivered the wrong visitors.

### Reading a SERP for winnability

| What you see at #1–3 | What it means |
|---|---|
| Registrars, marketplaces, or platform docs | Navigational/transactional intent you can't satisfy — different job, walk away |
| DR90+ publishers with dedicated pages | 24–36 months, and only with a link program |
| A thin Medium or Quora post | **Winnable in months.** Nobody has committed to this query |
| A dead or redirected page | Orphaned SERP — the incumbent left. Take it |
| Nothing on-topic at all | Vacant. This is a land grab, see below |
| Two-to-three brand-new sites already ranking | Proven winnable by low-authority sites — the strongest possible signal |

### The operational head term

Instead of the category noun, pick the **narrowest phrase that still contains your actual buyer.** For customdomain.ai that was **"custom domains for SaaS"** — real commercial intent, and the then-#1 was a low-DR site. Verdict: top-3 achievable in 6–12 months by deepening the pillar page rather than by chasing authority (estimate, unmeasured).

The local equivalent, from the lawn-care engagement: **avoid head terms like "lawn care `<city>`"** — they are dominated by national aggregators (LawnStarter, Lawn Love, Angi) whose entire business is ranking for them. The stated posture was explicit: win long-tail local, cost, seasonal, and hyperlocal queries where competitors are absent, especially on AI-answer surfaces.

Both baselines make the same point in numbers ([Search Console](search-console.md#baseline-discipline-record-before-changing)):

| Property (90d, 2026-07) | What the query rows said |
|---|---|
| customdomain.ai | Average position 42.5. Impressions on exactly the right intents at unrankable positions — "custom domain" ~72, "use your own domain" ~58, "saas domain names" ~45 |
| headsupoutdoorservices.com | Average position 14.5 — but that average was almost entirely the brand term at ~1.8. Commercial terms were nowhere: "grass cutting service" 27.2, "dethatching shakopee" 45 |

Position 40–70 is Google saying "topically relevant, authority not earned." No amount of on-page tuning moves that. Changing *which query you're competing for* does.

## Vacant niches: where the wins actually are

A vacant niche is a query with **real intent, weak or absent incumbents, and a genuine match to what you do.** All three, or it isn't one. Miss the third and you win a term that sends you nobody.

The live-SERP triage from the customdomain.ai program (verdicts as of 2026-07; timeframes are estimates, and the outcomes are **unmeasured** in the source window):

| Query family | Why it was winnable | Estimated window |
|---|---|---|
| "manage client domains" | #1 was a thin Medium post; agency buyer with real budget | Months — the most winnable family found |
| "MCP for domains" | **Completely vacant** — no committed page existed; first-mover window closing | 2–6 weeks with a repo + landing page + [registry listings](../agents/mcp-registry.md) |
| "domain connection api" | Orphaned SERP — the #1 result was a dead page | Months |
| "connect domain widget" | Unclaimed category noun; winnable with a live embedded demo | Months |
| "one click dns setup" | #1–2 were thin single-integration posts | Months |
| "custom domain vs subdomain" | A Quora answer at #1 — the single most beatable generic SERP found | 3–6 months |
| "`<competitor>` alternative" | Three brand-new sites already ranked — proof of low difficulty | Months, with a real wedge |

Adjacent plays that come free with the same research: **naming the category yourself** (publishing the liftable one-sentence definition of "custom domains as a service"), **matrix pages** (registrar × platform combinations), and **myth-busting pages** for a widespread misconception in your field. Category-definition pages are disproportionately valuable for [AI citation](../ai-search/content-that-gets-cited.md) because assistants need a definition to quote.

!!! tip "The agent layer is the emptiest niche of all"
    "MCP for `<category>`" queries were vacant across the board in mid-2026. If you have an API, the [MCP registry](../agents/mcp-registry.md) plus one landing page is the cheapest first-mover position in this book.

## The scoring framework

Turn the research into a ranked list rather than an argument. The repeatable loop:

1. **Mine Search Console for striking distance.** Query rows at **positions 8–20** are terms Google already considers you relevant for. These are the cheapest wins in existence — usually one better page away.
2. **Expand the seeds.** Autocomplete, People Also Ask, related searches, and the phrasings people actually use in community threads (Reddit, Hacker News, Stack Overflow, category-specific forums).
3. **Run the competitor gap.** What do the sites already ranking cover that you don't — and, more usefully, what does *nobody* cover?
4. **Cluster by intent**, not by string similarity. "custom domain vs subdomain" and "should I use a subdomain" are one page.
5. **Score each cluster:**

    ```text
    Score = (Demand × Relevance × BusinessValue) ÷ Difficulty
    × 1.5   if the term is already in striking distance (GSC positions 8–20)
    park it if Difficulty ≥ 4 (revisit in 6 months)
    ```

    Rate each factor 1–5. **Relevance and BusinessValue are the veto factors** — a high-demand term you can't serve scores itself out. Parking hard terms is the discipline that keeps a 90-day plan honest.
6. **Map one cluster to one page.** Two pages for one intent cannibalize each other; one page for two intents ranks for neither.
7. **Re-run monthly against Search Console.** New striking-distance rows appear as pages age; that's your next month's queue.

The three-question gut check, if you want it without arithmetic: *Is there real intent behind this query? Is the incumbent weak? Does it match what we actually do?* Anything short of three yeses is a maybe, and maybes get parked.

## Intent-keyword research for GEO

Answer engines don't match keywords, they match *questions* — so the research artifact is a list of literal phrasings, not a list of terms. Treat **query phrasing as an inventory to own**: each phrasing becomes an H2, and the sentence beneath it is the answer an assistant can lift verbatim.

Enumerate by awareness stage:

| Stage | What the query sounds like |
|---|---|
| **Problem-aware** | "SSL cert stuck pending", "how do I let users use their own domain" |
| **Solution-aware** | "custom domains as a service", "managed custom domain solution", "build vs buy" |
| **Vendor-aware** | "`<product>` alternative", "best `<category>` tools 2026" |
| **Dev / agent** | "API to connect a domain", "MCP server for domains", "can an AI agent set up a custom domain without a human" |

Build the map by segment → job-to-be-done → literal queries. A condensed version of the one that drove the SaaS program:

| Segment | Job to be done | Literal queries |
|---|---|---|
| PLG SaaS founder | Let customers use their own domain | "how do I let users use their own domain on my SaaS" |
| Platform/infra engineer | HTTPS for thousands of customer domains | "on-demand TLS for SaaS", "Let's Encrypt rate limits custom domains" |
| Agency / dev shop | Repeatable client-domain setup | "managed custom domain solution", "manage client domains" |
| AI-agent builder | Agent provisions a domain for the app it just built | "MCP server to connect a domain" |

Two research habits that matter more than volume data:

- **Check who *ranks* and who *AI cites* separately.** They are frequently different sets. Ranking tells you the SEO fight; citations tell you which sources answer engines trust for that intent — usually third-party (reported: ~85–93% of AI citations are third-party, Reddit around 40% of them). If citations are all off-site, your winning move may be [off-site presence](../ai-search/offsite-signals.md), not another page.
- **Write the answer before the page.** If you can't state the 40–60-word answer to the query, the page isn't ready. That answer becomes the opening block; details follow.

## The 90-day battle plan

Ship the plan as three dated waves, each with a verification step. The real one from the SaaS program (2026-07):

**Weeks 1–2 — fix the foundation and grab what's vacant**

- Brand-entity sprint (Organization schema + `alternateName` + `sameAs`, profile consistency, one-token brand writing)
- The vacant-niche land grab: repo + landing page + registry/directory listings for the term nobody owns
- The first comparison page
- Docs indexability fixes and the JobPosting markup repair (defects block everything downstream)

**Weeks 3–6 — one page per winnable niche**

- One page per validated vacant/orphaned query family, each answer-first with question-shaped H2s, a comparison table, and a `FAQPage` graph
- Every page pushed via [IndexNow](../bing/indexnow.md) on publish and reflected in the sitemap

**Weeks 7–12 — depth and authority**

- Canonical developer guides and matrix pages (the pages that earn links on their own)
- A deliberate link wave: launch posts, marketplace and directory listings, podcasts. Target for year one was 100–300 category-relevant referring domains
- Monthly Search Console re-mine to refresh the striking-distance queue

**Execution proof, and its honest caveat:** in one day this plan produced 15 new pages (3 comparison, 3 guides, 4 category/landing, 5 glossary) plus a listicle and a keyworded homepage H1 — the sitemap went 93 → 101 → 117 URLs across the waves, resubmitted to Search Console each time. **Use the sitemap URL count as your execution ledger.** The ranking outcome of those pages is **unmeasured**: the measurement window closed before the results were in, and this book does not round up.

## Content allocation: money pages, cost guides, seasonal

Where the pages actually go. The split below is the local-business version (from the lawn-care program's content plan); the SaaS analogue follows.

**Money pages — roughly ten.** One dedicated page per real service, not per bundle: 800–1,500 words covering what's included, the process, what makes you different, genuinely local specifics (soil type, regional grass species, county ordinances), a real price band, two or three real photos, an FAQ block, and two or three **real review snippets for that specific service**. Each carries `Service` schema with `areaServed`. This is where commercial intent converts; build these before anything else.

**Cost guides — the highest-value asset most competitors skip.** "How much does snow removal cost in `<city>` — per-visit vs full-season", "cost of a paver patio in the `<region>`". Rules that make them work:

- **Lead with a dollar band and the locale in the first sentence.** That sentence is what an answer engine lifts.
- **Only real ranges, from the business.** Never invent a price. Our version was built exclusively by doing arithmetic on prices already published on the site.
- **Prices must match everywhere** — page copy, price list, and schema. Conflicting prices across pages are an SEO defect, not just a copy defect; one audit found FAQ markup effectively telling Google the prices didn't exist while pages showed different numbers.
- **Consolidation is a legitimate endgame.** The owner later retired the cost guides *into* the pricing page with named 301 redirects. That's consolidation, not loss — one strong page beats five thin ones.

**Seasonal engine — three pillars, each an interlinked cluster.** "Month-by-month lawn calendar for `<climate zone>`", "first-snow prep for `<city>` driveways". **Publish ahead of the season, not during it**: freshness is a real retrieval factor for some answer engines (Perplexity reportedly re-crawls fresh content within days), and seasonal queries spike before the season starts.

**The SaaS analogue** of the same allocation: a pillar page for the operational head term, a glossary that owns the category vocabulary (`DefinedTerm` per entry), per-use-case pages, developer how-to guides (`TechArticle` + `FAQPage`), and comparison pages if brand policy allows them. Hub-and-spoke internal links plus footer links to the cluster give it sitewide link equity. Details in [content that gets cited](../ai-search/content-that-gets-cited.md).

**What not to build, from real teardowns:**

- **Doorway pages.** Twenty-two thin street- and neighborhood-level pages, linked in every footer, were unlinked, 301'd into their city pages, and deindexed. Thin geographic pages are a liability with Google's spam updates and with anyone reading them.
- **City-swap templates.** An audit measured city-page copy at 72% similarity with the city name blinded, with several sections byte-identical. If you can't say something genuinely specific about a city, don't publish a page for it ([service-area pages](../local/service-areas.md)).
- **Programmatic location × service farms.** The market incumbent ran roughly 5,900 generated pages. For a single operator that's thin content at scale — compete on specificity instead.

## Gotchas

- **A branded search that returns nothing is a strategy problem, not a bug.** No amount of on-page work fixes tokenization; only an entity program does, over months.
- **Writing your brand as spaced generic words externally** actively trains engines against you. One token, everywhere.
- **Trusting a tool's difficulty score over the live SERP.** A "high difficulty" term whose #1 is a Quora answer is winnable; a "low difficulty" term held by three DR90 domains is not.
- **Chasing the category noun.** High volume, wrong intent, three-year timeline. Take the narrower phrase that contains your actual buyer.
- **Ignoring striking distance.** Positions 8–20 in Search Console are the cheapest wins you will ever have, and most teams never look.
- **Winning a term you can't serve.** Relevance and business value are veto factors, not weights.
- **One intent, two pages** — cannibalization. Map clusters to pages deliberately.
- **Publishing seasonal content in season.** Too late for both crawling and buyers.
- **Inventing numbers to fill a cost guide.** A fabricated price is a fabricated claim; it fails the same [authenticity standard](../local/authenticity.md) as a fake review, and it becomes a commitment when a customer quotes it back.
- **Declaring victory before measuring.** Record the [baseline](../foundations/measurement.md) first, re-measure at 2–4 weeks, and say "unmeasured" when it is.

## Related

- [Search Console](search-console.md) — striking-distance mining and the baseline discipline this chapter runs on
- [Google](index.md) — which lever moves which Google surface
- [GEO fundamentals](../ai-search/geo-fundamentals.md) — the branded-vs-intent reframe across all three surfaces
- [Content that gets cited](../ai-search/content-that-gets-cited.md) — how to write the pages this plan schedules
- [Entities, E-E-A-T, and trust](../foundations/entities-and-trust.md) — the entity program the tokenization fix depends on
- [Off-site signals](../ai-search/offsite-signals.md) — when the citations live somewhere other than your site
- [GitHub as a discovery surface](../agents/github-as-discovery.md) — the same whitespace logic applied to GitHub search
- [Service-area pages](../local/service-areas.md) — the honest version of local page building
- [Case study: customdomain.ai](../case-studies/customdomain-ai.md) — the tokenization crisis in its full context

Source skills: [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization) · [positioning-basics](https://github.com/ever-just/agentskills/tree/main/skills/white-paper-writing/ai-marketing-skills/positioning-basics) · [homepage-audit](https://github.com/ever-just/agentskills/tree/main/skills/white-paper-writing/ai-marketing-skills/homepage-audit)
