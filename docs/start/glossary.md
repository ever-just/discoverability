# Glossary

Working definitions as this book uses them. Terms link to the chapter that treats them in depth.

**AEO (Answer Engine Optimization)** — Optimizing to be *the cited answer* in AI-generated responses (AI Overviews, ChatGPT, Perplexity). Overlaps GEO; AEO emphasizes question-shaped queries and answer-first content. → [AI Search](../ai-search/index.md)

**AI Overviews / AI Mode** — Google's AI-generated answer layers. Overviews appear above classic results; AI Mode (`udm=50`) is the fully conversational surface. Both draw on Google's index. → [How discovery works](how-discovery-works.md)

**Answer engine** — A system that returns one synthesized answer with citations instead of a ranked list of links.

**Answer-first content** — Page structure where the direct answer appears in the first block, self-contained enough for an LLM to quote, with elaboration after. → [Content that gets cited](../ai-search/content-that-gets-cited.md)

**Baseline** — The metrics snapshot (clicks, impressions, position, AI-visibility spot checks) recorded *before* making changes, without which you can't attribute results. → [Measurement](../foundations/measurement.md)

**Bot challenge** — A WAF/CDN interstitial (CAPTCHA, JS proof-of-work, 429) served to suspected bots. When it fires on Googlebot or GPTBot, your site silently vanishes from search and AI answers. → [Rendering & WAFs](../technical/rendering-and-waf.md)

**Canonical URL** — The URL you declare as the authoritative version of a page (`rel=canonical`). Reverse proxies that rewrite the Host header are notorious for emitting canonicals pointing at internal hostnames. → [Reverse proxies](../technical/reverse-proxy-cms.md)

**Citation** — An answer engine's attribution of a claim to a source, usually a link. The GEO equivalent of a ranking. → [GEO fundamentals](../ai-search/geo-fundamentals.md)

**Crawl budget** — How much a crawler is willing to fetch from your site per unit time. Mostly a large-site concern; small sites' problem is usually crawl *permission*, not budget.

**Crawlability** — Whether a bot can fetch and read your pages at all: robots permissions, no challenges, server-rendered content. The gate every other tactic sits behind. → [AI crawlers](../ai-search/ai-crawlers.md)

**DMARC / DKIM / SPF** — The three DNS-based email authentication standards. Together they decide whether your mail lands in inboxes; they're also a trust signal for your domain generally. → [Email trust](../technical/email-trust.md)

**DNS-AID** — An emerging pattern for resolving a domain to its agent/MCP endpoint via DNS records (SVCB + TXT + `_index._agents` + TLSA/DANE under DNSSEC), giving agents a DNS-native trust path. → [Manifests and DNS-AID](../agents/manifests-and-dns.md)

**Domain property (sc-domain)** — A Search Console property type covering every protocol/subdomain of a domain, verified via DNS. Required for whole-domain visibility; supports service-account API access. → [Search Console](../google/search-console.md)

**E-E-A-T** — Experience, Expertise, Authoritativeness, Trustworthiness: Google's framework for evaluating source quality. Not a metric you set — an impression your evidence earns. → [Entities and trust](../foundations/entities-and-trust.md)

**Entity** — A distinct "thing" (organization, person, product, place) that engines model beyond keywords. Consistent structured data and `sameAs` links teach engines your entity. → [Entities and trust](../foundations/entities-and-trust.md)

**GEO (Generative Engine Optimization)** — Optimizing so generative AI systems cite your property for category and intent queries, not just brand queries. → [GEO fundamentals](../ai-search/geo-fundamentals.md)

**GBP (Google Business Profile)** — Google's local business listing system; feeds the map pack, local knowledge panels, and local AI answers. → [Business Profile](../google/business-profile.md)

**GPTBot / OAI-SearchBot / ChatGPT-User** — OpenAI's three crawlers: training-data collection, search indexing, and live user-initiated fetches respectively. Each honors robots.txt separately. → [Crawler registry](../appendix/crawler-registry.md)

**Host-rewriting proxy** — A reverse proxy that changes the `Host` header before your app sees it (common in multi-tenant platforms). The root cause of a whole family of robots/canonical/sitemap bugs. → [Reverse proxies](../technical/reverse-proxy-cms.md)

**IndexNow** — A push protocol (adopted by Bing and others) that notifies engines of changed URLs instantly instead of waiting for a crawl. → [IndexNow](../bing/indexnow.md)

**Indexing API (Google)** — Google's push API, officially scoped to JobPosting and BroadcastEvent pages only — not a general "index my site faster" tool. → [Search Console](../google/search-console.md)

**Intent query** — A search expressing a need ("connect custom domains to my SaaS") rather than a name ("customdomain.ai"). The valuable, contested half of discovery. → [Keyword strategy](../google/keyword-strategy.md)

**JSON-LD** — The recommended syntax for schema.org structured data: a `<script type="application/ld+json">` block, independent of your visible HTML. → [Structured data](../google/structured-data.md)

**`@graph` / `@id`** — JSON-LD mechanics for shipping multiple cross-referencing nodes in one block; `@id` lets a page's `Service` point at the sitewide `LocalBusiness` node without repeating it. → [LocalBusiness schema](../local/local-business-schema.md)

**llms.txt** — A proposed markdown index file for LLM consumption. Reality check: most deployments get zero bot requests and Google says it doesn't use it; it earns its keep mainly on docs sites read by coding agents. → [llms.txt](../ai-search/llms-txt.md)

**Map pack** — The 3-result local block in Google results, driven by GBP data, proximity, and reviews rather than classic page ranking.

**MCP (Model Context Protocol)** — The open protocol AI agents use to discover and call tools. Your MCP server is a product surface for agents the way your website is for humans. → [AI Agents](../agents/index.md)

**MCP Registry** — `registry.modelcontextprotocol.io`, the canonical listing of MCP servers, mirrored by community directories (mcp.so, Smithery, Glama, PulseMCP). → [MCP Registry](../agents/mcp-registry.md)

**NAP consistency** — Name, Address, Phone appearing identically everywhere (site, GBP, directories). Inconsistency fragments your local entity.

**Noindex** — A directive (meta tag or header) telling engines not to index a page. Several CMS settings set it *implicitly* — the trap catalog is longer than you'd think. → [Sitemaps and robots](../google/sitemaps-and-robots.md)

**OfferCatalog** — The schema.org structure for a tiered pricing catalog. Must hang off a `Service` node (`hasOfferCatalog`) — not `SoftwareApplication`. → [Rich results](../google/rich-results.md)

**Answer-time retrievability** — Whether an engine can fetch your page *at the moment it composes an answer*. Distinct from being indexed; blocked live-fetch bots (ChatGPT-User, Claude-SearchBot) kill citations even for indexed pages. → [AI crawlers](../ai-search/ai-crawlers.md)

**Rich results** — Enhanced result presentations (stars, FAQs, software cards, breadcrumbs) earned by specific schema types passing eligibility rules. → [Rich results](../google/rich-results.md)

**robots.txt** — The crawler permission file at your domain root. The first thing to check in any invisibility investigation — including whether the *served* file matches the one you think you deployed. → [Sitemaps and robots](../google/sitemaps-and-robots.md)

**sameAs** — The schema.org property linking your entity to its profiles elsewhere (GitHub, LinkedIn, social, registries). How engines consolidate your identity. → [Entities and trust](../foundations/entities-and-trust.md)

**Search grounding** — An AI assistant running live web searches to inform its answer. The dominant route to being cited today. → [How AI finds and cites](../foundations/ai-retrieval.md)

**SERP** — Search Engine Results Page.

**Service-area page** — A local page targeting "service + city". Legitimate when the business truly serves that city with evidence; a spam pattern when mass-produced. → [Service-area pages](../local/service-areas.md)

**server.json** — The manifest describing an MCP server for the registry: name (reverse-DNS namespace), endpoints, auth, description. → [MCP Registry](../agents/mcp-registry.md)

**Tokenization crisis** — When a brand name made of generic words (e.g. a dotted domain like "customdomain.ai") gets split into its parts by the engine, making even branded search fail. → [Keyword strategy](../google/keyword-strategy.md)

**UnitPriceSpecification** — The schema.org structure for recurring prices ("$249 per month" as `price` + `referenceQuantity` with `P1M`). → [Rich results](../google/rich-results.md)

**URL Inspection API** — Search Console's per-URL diagnostic API: index status, detected canonicals, structured-data errors, as Google sees them. → [Search Console](../google/search-console.md)

**Vacant niche** — A query space with real intent and no committed owner — winnable in weeks while head terms take years. The core of realistic keyword strategy. → [Keyword strategy](../google/keyword-strategy.md)

**`.well-known`** — The URL path convention (`/.well-known/...`) for machine-readable discovery documents: OAuth metadata, agent manifests, security.txt. → [OAuth discovery](../agents/oauth-discovery.md)
