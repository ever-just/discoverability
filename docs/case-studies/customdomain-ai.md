# customdomain.ai — the SaaS case study

customdomain.ai is a domain-connection product ("custom domains as a service": automated DNS configuration, on-demand TLS, a REST API, and an MCP server for AI agents). Its marketing site runs on a CMS tenant behind a Host-rewriting reverse proxy; its docs and app live on a subdomain; its agent surface is a separate service. Over three weeks in July 2026 it went from a freshly renamed product on an effectively invisible domain to a property with validated rich-result schema, a Bing→ChatGPT indexing pipeline, an official MCP Registry listing, and a keyword strategy built around niches it can actually win. This page is the dated record — including the audit that found the product's differentiated path quietly broken, and the numbers that are still unmeasured.

!!! abstract "What this proves"
    - **The two-user reframe.** A modern SaaS product has two distinct users to be discovered by: the human buyer (won via search + AI answers) and the AI agent told to *use* the product (won via registries, discovery endpoints, and tool design). They need different work, in different codebases, and neglecting the second silently forfeits the greenfield.
    - **Discoverable is not usable.** Every agent-discovery surface was formally live — and an adversarial audit still found the agent path broken at the code level. Being found is necessary; letting the agent *finish* is the product.
    - **Verify at the layer that serves users.** The single recurring failure mode: a change verified in the database, the docs, or a memory note that was not true in the rendered HTML, the live SERP, or the running binary.
    - **Baseline before you build.** The Search Console baseline (5 clicks / 159 impressions / position 42.5) was captured *before* the content program, so future movement is attributable. Everything not yet re-measured is listed as unmeasured below.

## The starting problem

Three compounding disadvantages, none obvious from inside the product:

1. **A brand Google cannot see.** Searching `customdomain.ai` returned zero results for the site — Google tokenizes the name into "custom domain" + "ai" and serves registrar and AI-website-builder pages. Branded discovery, the supposedly easy half, was broken on day one ([the tokenization crisis](../google/keyword-strategy.md)).
2. **A young domain in a category with no settled name.** "SSL for SaaS", "CDaaS", "managed custom domains" — when the category vocabulary is contested, you have to publish the definition you want quoted.
3. **Two audiences, two surfaces, one team.** The findability work (marketing site, schema, Bing) and the agent-usability work (MCP tools, OAuth, error payloads) live in different codebases with different failure modes.

## Timeline

### 2026-07-01 → 07-04 — the product sprint

The product is built in a ~3-day sprint as an open-source alternative in the domain-connection space: DNS auto-configuration, on-demand TLS, a reverse-proxy edge, multi-tenant from day one. Discoverability is not yet a workstream — which is normal, and is exactly why the rest of this timeline exists.

### 2026-07-05 → 07-06 — the surface split

Marketing moves from a Next.js app to a CMS tenant at the apex; the product console and docs move to a subdomain. Each host gets its own robots.txt, sitemap, and llms.txt. Lesson that recurs all month: **every host is its own discovery surface** with its own crawlability, canonical, and verification state.

### 2026-07-10 — the first crawlability crisis, found by accident

An ["Ask AI about us" deep-link row](../ai-search/ask-ai-widget.md) ships in the site footer (five verified providers; Gemini and Copilot skipped — their `?q=` schemes were broken as of 2026-07). The real payoff is the failure it exposes:

!!! warning "The site was telling every crawler to go away"
    A real click-test showed ChatGPT *searching* for the product (via Bing) instead of fetching the URL — and finding nothing, because the site was unindexed. Probing as bot user-agents found why: robots.txt served `Disallow: /`. The CMS's robots template compares the request host to the configured site domain, and behind the Host-rewriting proxy that comparison can never succeed. The server returned 200 to every AI fetcher; robots.txt told them all to leave. Fixed the same day by overriding the template clause. The full bug class this belongs to is cataloged in [the everjust-tenants case](everjust-tenants.md) and [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md).

A second lesson from the same test: the v1 widget prompt said "read our llms.txt first", which made ChatGPT search for *articles about the llms.txt format*. The fix — bake accurate product facts into the prompt itself, self-contained, URL only as a footnote — became the [self-contained-prompt rule](../ai-search/ask-ai-widget.md).

### 2026-07-11 — the Bing pipe and the content cluster

Because [Bing gates ChatGPT search](../bing/index.md), the day's work is Bing-first: site verification shipped two ways (root verification file served from the proxy edge — app-served files 404 during deploys, precisely when the verifier clicks — plus the meta tag), [IndexNow](../bing/indexnow.md) live with 73 URLs accepted (HTTP 202), then automated (a 6-hour sweep of the live sitemap, plus a 4-hour job clearing the CMS's cached sitemap so new pages actually appear). Research verdicts recorded the same day and never reversed: [llms.txt is not an AI-search lever](../ai-search/llms-txt.md) (~97% of files get zero bot requests; its real consumer is coding agents), Google's Indexing API is off-label for marketing pages, and the sitemap-ping endpoint has been dead since 2023.

The internal tenant hostname is discovered to be a fully crawlable duplicate of the whole site — fixed with a noindex header on that host (never a redirect; backend links depend on it) and proxy-level rewriting of the sitemap to canonical URLs.

The first [content cluster](../ai-search/content-that-gets-cited.md) ships: an answer-first pillar page (11 question-shaped H2s, FAQPage + SoftwareApplication JSON-LD), four glossary definition pages, three developer guides, hub-and-spoke internal links, and a footer "Resources" column for cluster-wide link equity.

### 2026-07-12 — the agent-discovery layer

The MCP server (already built: 11 tools, streamable HTTP) gets its discovery plumbing in one push: the [OAuth discovery chain](../agents/oauth-discovery.md) (401 responses now carry a `resource_metadata` pointer; RFC 9728 and RFC 8414 well-knowns live), a [capability manifest](../agents/manifests-and-dns.md) at `/.well-known/agent/mcp.json`, publication to the official [MCP Registry](../agents/mcp-registry.md) via DNS-verified namespace claim, DNS-AID records, and DNSSEC. Sequencing rule learned en route: **endpoint first, records second** — discovery pointers at a dead endpoint are hollow.

### 2026-07-15 — the deep audit: the differentiated path was quietly broken

A simple question ("did we actually submit to Bing?") escalates into a live-probed scorecard of seven portfolio properties (finding: only customdomain.ai was Bing-verified; one sister property's sitemap contained a single URL), then into a full three-pillar audit — findability, agent discovery *and usability*, and ICP phrasing — that produces the two-user reframe and ten critical findings.

!!! warning "The headline findings (all verified, all fixed)"
    - **The MCP dropped the DNS records.** The control plane returned the authoritative record set for every connection; the MCP's conversion layer discarded it. An agent literally could not tell its user which CNAME to add — defeating the product's core thesis.
    - **The advertised terminal status never arrives.** Docs said connections finish as `completed`; the backend's real terminal status was `live`. Agents polling for the documented value would poll forever. Fix: correct the enum and expose an explicit boolean "done" signal.
    - **Prod-vs-source drift.** The OAuth well-knowns, capability manifest, and registry metadata were live in production but existed nowhere in the repository — a rebuild would silently erase the entire discovery layer. Fix: version them in code with tests asserting served == advertised.
    - Plus: no tool annotations (hard-gating the first-party connector directories), no credential path for a cold-start agent (discovery chain ends at a door with no handle), an llms.txt that never mentioned the MCP, and docs contradicting reality (6 vs 11 vs 12 tools).

    The fix slice ships the same afternoon: records surfaced in tool results, corrected status semantics, a twelfth `list-connections` tool, [annotations on every tool](../agents/tool-descriptions.md), and server-level `instructions` that teach the whole model — including the one human step (the user, not the agent, writes DNS records at their provider).

    A process note that generalizes: three of the ten findings had *already moved* by implementation time (the codebase kept evolving during the audit). Point-in-time audits rot in days — reconcile against the changelog before fixing anything.

The same day, Search Console access via a service account produces the program's ground truth: **5 clicks / 159 impressions / average position 42.5 over 90 days — and the sitemap had never been submitted** (submitted via API that day). Live SERP recon turns the keyword program honest: the generic head term is judged unwinnable for 24–36 months, and effort moves to [vacant and winnable niches](../google/keyword-strategy.md) — the operational head term ("custom domains for SaaS"), orphaned SERPs ("manage client domains", "domain connection api"), and a genuinely vacant first-mover niche ("MCP for domains", estimated winnable in weeks). Fifteen intent-targeted pages ship in one day; the sitemap grows 101 → 117 URLs and is resubmitted.

### 2026-07-16 — the GitHub org funnel

GitHub becomes a deliberate discovery surface: org profile README as the front door, a public docs repo as source of truth, audience-targeted repos with keyword-dense descriptions and exact-match topics (most category topics were unclaimed), an awesome-list created for the category because none existed, and AGENTS.md files for the agents that auto-read them. Grounding facts: GitHub repo search indexes only name/description/topics, and Bing has a privileged GitHub crawl — public repo content reaches ChatGPT through it. Details in [GitHub as a discovery surface](../agents/github-as-discovery.md).

A same-week audit grades the agent surface "top few percent of MCP servers" on discovery plumbing — and grades **observability F**: Bing Webmaster Tools was still unverified, meaning zero visibility into the one place that reports AI/Copilot citations. Running the pipeline blind was named the cheapest, highest-leverage remaining fix.

### 2026-07-19 — the maximal schema graph, and the view-routing trap

The schema program ships the book's flagship structured-data example: a homepage `@graph` of 11 cross-linked nodes — dual **SoftwareApplication + Service** (the software node is the only one Google renders a software rich result for; the Service node exists because `hasOfferCatalog` is not valid on SoftwareApplication), an 18-item `featureList`, an OfferCatalog of the five real pricing tiers ($0 / $249 / $749 / two custom — custom tiers get *no* price node, because `"0"` would read as free), a DefinedTermSet glossary, and `potentialAction` EntryPoints advertising the REST API, the OpenAPI document, and the MCP endpoint — machine-readable signposts from the human surface to the agent surface. Design detail in [Structured data](../google/structured-data.md) and [Rich results](../google/rich-results.md).

The pricing page renders the new block instantly. The homepage refuses to.

!!! warning "The view-routing trap"
    The homepage block was injected, verified present in the database, and never appeared in the rendered HTML — through a cache-theory detour and a full service restart. The root cause: the site's homepage setting pointed `/` at a *different page's template* than the one the tooling mapped to `url: "/"`. The schema sat, correct and "verified", in a template that no longer served the route. Diagnosis that worked: prove *which template actually renders the route* with a distinctive-string test before blaming caches. Two nested traps en route: injecting into an editor-managed container that the renderer strips, and a write path that doesn't invalidate warm workers' compiled templates. Full pattern in [the everjust-tenants case](everjust-tenants.md).

### 2026-07-20 → 07-21 — validation and the entity file

Roughly 30 hours after the schema design session, Google's Rich Results Test validates both pages live: homepage "3 valid items", pricing "2 valid items", zero errors — the only notes being an intentionally omitted `aggregateRating` (no real on-page reviews yet; fabricating one is a spam violation). The nuance recorded for posterity: the Rich Results Test only reports Google's *visual* rich-result types; the Service/OfferCatalog/DefinedTerm payload parses fine and is aimed at answer engines and the Knowledge Graph, not at SERP decoration.

Same window: an existing unverified Google Business Profile is found and completed (the Business Profile *API* was the wrong tool — allowlist-gated and bulk-oriented; a signed-in browser session was the practical lane), with final verification left to the owner. A human-in-the-loop agent-authorization flow (dynamic client registration → PKCE → scoped consent) goes live in production, closing the audit's "discovery chain ends at a door with no handle" finding.

## The numbers

| Metric | Value | Date | Status |
|---|---|---|---|
| GSC 90-day baseline | 5 clicks / 159 impressions / avg position 42.5 | 2026-07-15 | Measured, pre-program |
| Branded SERP for `customdomain.ai` | 0 results for the site | 2026-07-15 | Measured |
| Sitemap on first GSC inspection | 101 URLs, never submitted | 2026-07-15 | Measured |
| Sitemap after the content program | 117 URLs (93 → 101 → 117) | 2026-07-15 | Measured |
| First IndexNow submission | 73 URLs, HTTP 202 | 2026-07-11 | Measured (submission ≠ citation) |
| Homepage schema graph | 11 nodes, 18-item featureList | 2026-07-19 | Live, third-party validated |
| Rich Results Test | 3 valid items (home) + 2 (pricing), zero errors | 2026-07-20 | Measured |
| Design → validated rich-result eligibility | ~30 hours | 2026-07-19 → 07-20 | Measured |
| MCP tools | 12 (docs had claimed 6; code had 11) | 2026-07-15 | Measured |
| Intent pages shipped in one day | 15 | 2026-07-15 | Measured |
| New-domain indexing expectation set | first pages ~2–4 weeks; fuller ~2–3 months | 2026-07-11 | Documented expectation, not a result |

**Still unmeasured, honestly:** post-program GSC movement (the re-measure window postdates this record); AI-citation share for the ICP query set (the weekly ChatGPT/Perplexity/AI-Mode battery was designed, with its own rule: "everything above is a hypothesis until measured"); Bing/Copilot reception (Bing Webmaster Tools was still unverified at record end — the graded-F observability gap); MCP registry referral and connect rates. The plan's own words apply: shipping is not the outcome.

## What worked

- **Bing-first sequencing.** One verification + IndexNow unlocked the index that feeds ChatGPT search, for hours of effort.
- **The dual-node schema design.** Working *with* the vocabulary's constraints (OfferCatalog on Service, rich results on SoftwareApplication) instead of fighting them produced a graph that validated on the first live test.
- **Live-SERP triage before writing content.** Fifteen pages aimed at verified-winnable niches beat any volume of content aimed at a DR-90 head term.
- **The audit posture.** Treating every prior claim as a hypothesis until live-verified is what surfaced the broken agent path, the unsubmitted sitemap, and the prod-vs-source drift.
- **Baseline discipline.** Ten minutes of Search Console API work made every future claim about the program falsifiable.

## What failed

- **The differentiated path was broken while every box was checked.** Registry: listed. OAuth: discoverable. Manifest: live. And an agent still couldn't complete the core job. Surface-level "shipped" hid code-level "unusable".
- **The brand name itself.** A dotted generic-word brand produced a zero-result branded SERP; the entity-repair program (one-token brand writing, consistent profiles, `sameAs` graph) is a months-long tax that a different name would never have paid.
- **The v1 Ask-AI prompt** sent users to a search about a file format. Real-user testing caught in minutes what design review missed.
- **"Verified" that wasn't.** The homepage schema was verified in the database while the rendered page went without it — the view-routing trap cost hours and a restart.
- **Observability lagged shipping.** Weeks of pipeline work ran blind because the one dashboard that shows AI citations (Bing Webmaster Tools) was never verified during the window.

## What we'd do differently

- **Name-check the brand against tokenization before committing to it** — one incognito search would have predicted the crisis.
- **Verify Bing Webmaster Tools the same day as IndexNow**, not weeks later. Submission without reception data is motion, not progress.
- **Ship tool annotations and the credential path with the first MCP release.** Both were hard gates for first-party directories and for real agent autonomy; both were retrofits.
- **Put "prove which template serves the route" first** in any CMS debugging checklist, ahead of every cache theory.
- **Schedule the re-measure when you ship the change.** Several outcomes are unmeasured above because the measurement date was left implicit.

## Chapters this case feeds

- [Keyword and SERP strategy](../google/keyword-strategy.md) — the tokenization crisis and the vacant-niche battle plan
- [Structured data](../google/structured-data.md) and [Rich results](../google/rich-results.md) — the dual-node @graph as the worked example
- [Google Search Console](../google/search-console.md) — service-account access and baseline discipline
- [Bing & Beyond](../bing/index.md) and [IndexNow](../bing/indexnow.md) — the Bing→ChatGPT pipe
- [llms.txt — the reality check](../ai-search/llms-txt.md) and [The Ask-AI widget](../ai-search/ask-ai-widget.md)
- [The MCP Registry](../agents/mcp-registry.md), [OAuth discovery chain](../agents/oauth-discovery.md), [Manifests and DNS-AID](../agents/manifests-and-dns.md), [Tool descriptions that rank](../agents/tool-descriptions.md), [GitHub as a discovery surface](../agents/github-as-discovery.md)
- [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md) — via [the everjust-tenants case](everjust-tenants.md)
- [Launch a SaaS product](../playbooks/saas-launch.md) — this timeline, generalized into a runnable sequence
- Source skills: [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization), [mcp-server-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/mcp-server-discoverability), [llm-deeplink-widget](https://github.com/ever-just/agentskills/tree/main/skills/llm-deeplink-widget), [reverse-proxy-cms-indexing](https://github.com/ever-just/agentskills/tree/main/skills/reverse-proxy-cms-indexing) — full mapping in the [skill index](../appendix/skills-index.md)

---

*Record window: 2026-07-01 → 2026-07-21, with portfolio-level remediation continuing into August (see [everjust-tenants](everjust-tenants.md)). Later developments will be appended below with dates; the record above stays as written.*
