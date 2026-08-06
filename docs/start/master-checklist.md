# The master checklist

The whole book compressed into one list. Work top to bottom — the ordering is deliberate (measurement before changes, crawlability before content, foundations before amplification). Each item links to the chapter that explains it.

## 0 · Before you change anything

- [ ] Record your baseline: Search Console clicks/impressions/position, current AI-visibility spot checks → [Measurement](../foundations/measurement.md)
- [ ] Run the [30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) and file the scorecard

## 1 · Crawlability (the gate)

- [ ] `robots.txt` allows what you think it allows — fetch it from the **public** URL and read it → [Sitemaps and robots](../google/sitemaps-and-robots.md)
- [ ] Explicitly allow the AI crawler roster (GPTBot, OAI-SearchBot, ClaudeBot, Claude-SearchBot, PerplexityBot, Google-Extended) → [AI crawlers](../ai-search/ai-crawlers.md)
- [ ] Curl your site as Googlebot and GPTBot — no 403/429/challenge pages → [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md)
- [ ] Key content renders server-side (view-source shows your copy) → [Rendering](../technical/rendering-and-waf.md)
- [ ] If behind a reverse proxy/CDN: canonicals, robots, and sitemap all show the **public** host, and the internal host isn't a crawlable duplicate → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)

## 2 · Measurement plumbing

- [ ] Google Search Console verified (consider a domain property + service-account API access) → [Search Console](../google/search-console.md)
- [ ] Bing Webmaster Tools set up (import from GSC takes minutes) → [Bing Webmaster Tools](../bing/bing-webmaster-tools.md)
- [ ] XML sitemap live, referenced in robots.txt, and actually fresh → [Sitemaps](../google/sitemaps-and-robots.md)
- [ ] IndexNow key deployed if your platform supports it → [IndexNow](../bing/indexnow.md)

## 3 · Identity and structured data

- [ ] One canonical schema.org graph strategy — CMS-auto **or** hand-authored, never both on the same node → [Structured data](../google/structured-data.md)
- [ ] Organization/LocalBusiness node with `sameAs` links to every real profile → [Entities and trust](../foundations/entities-and-trust.md)
- [ ] The node types that earn rich results for your category, validated live → [Rich results](../google/rich-results.md)
- [ ] Local business: sitewide `LocalBusiness` `@id` graph + per-page Service/FAQ/Breadcrumb → [LocalBusiness schema](../local/local-business-schema.md)
- [ ] FAQ schema generated from **visible** page content only → [FAQ schema](../local/faq-schema.md)
- [ ] Review markup only on Service nodes, only with real reviews → [Reviews](../local/reviews.md)

## 4 · Local (if applicable)

- [ ] Google Business Profile claimed, complete, categorized; API quota requested early if you'll automate → [Business Profile](../google/business-profile.md)
- [ ] Live review sync (Places caps at 5 via API — design for it) → [Reviews](../local/reviews.md)
- [ ] Service-area pages pass the honesty tests: real city limits, real photos, real reviews per area → [Service-area pages](../local/service-areas.md)
- [ ] Authenticity audit passed — no stock photos captioned as real jobs, no invented testimonials → [Authenticity audits](../local/authenticity.md)

## 5 · Content for intent queries

- [ ] Keyword reality check: is your head term winnable, or do you need the vacant-niche strategy? → [Keyword strategy](../google/keyword-strategy.md)
- [ ] Pillar page + glossary + how-to cluster for your category, answer-first → [Content that gets cited](../ai-search/content-that-gets-cited.md)
- [ ] Hub-and-spoke internal linking wired → [Content that gets cited](../ai-search/content-that-gets-cited.md)
- [ ] Meta title/description on every page that matters → [Search Console](../google/search-console.md)

## 6 · AI-specific surfaces

- [ ] llms.txt shipped if cheap on your stack (know its limits — it's mostly for coding agents) → [llms.txt](../ai-search/llms-txt.md)
- [ ] Ask-AI deep-link widget considered for footer/docs → [Ask-AI widget](../ai-search/ask-ai-widget.md)
- [ ] Off-site presence on the surfaces AI retrieves: directories, comparison content, Reddit/forums, GitHub → [Off-site signals](../ai-search/offsite-signals.md)

## 7 · Agent layer (if you have an API/MCP server)

- [ ] `server.json` published to the official MCP Registry with a verified reverse-DNS namespace → [MCP Registry](../agents/mcp-registry.md)
- [ ] Listings mirrored to the community directories → [MCP Registry](../agents/mcp-registry.md)
- [ ] OAuth discovery chain returns correctly end-to-end (401 → `resource_metadata` → both `.well-known` docs) → [OAuth discovery](../agents/oauth-discovery.md)
- [ ] Tool descriptions written for agent intent, not marketing → [Tool descriptions](../agents/tool-descriptions.md)
- [ ] GitHub org discoverable: names/descriptions/topics carry your keywords → [GitHub as discovery](../agents/github-as-discovery.md)

## 8 · Trust infrastructure

- [ ] You know who actually serves your DNS (`dig NS`) → [Domains and DNS](../technical/domains-and-dns.md)
- [ ] SPF + DKIM + DMARC live and tested → [Email trust](../technical/email-trust.md)
- [ ] If you ever changed domains: 301s live, old domain retired properly, GSC told → [Domain migrations](../technical/domain-migration.md)

## 9 · Keep it running

- [ ] Operating cadence on the calendar: weekly checks, monthly review, quarterly re-audit → [Operating cadence](../playbooks/operating-cadence.md)
- [ ] Re-run the AI visibility audit on schedule → [AI visibility audit](../ai-search/ai-visibility-audit.md)
