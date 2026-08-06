# BLUEPRINT — the master prompt that builds this textbook

> This file is both the **standing specification** for humans maintaining this repo and a **runnable prompt** for an AI agent asked to build, extend, or rebuild the book. It was authored 2026-08-06 from a full sweep of the EverJust discoverability corpus (skills, session transcripts, project memories, plan docs). If you are an agent: read this whole file, then execute §9.

---

## 1. Mission

Build and maintain **a comprehensive, textbook-quality knowledge base about making things findable** — websites, products, local businesses, APIs, and MCP servers — across every discovery surface that matters in 2026:

1. **Search engines** — Google (Search Console, Business Profile, schema.org, rich results), Bing (Webmaster Tools, IndexNow), and the long tail.
2. **AI answer engines** — ChatGPT Search, Perplexity, Google AI Overviews/AI Mode, Claude, Copilot: getting **cited as the answer**, not just indexed.
3. **AI agents** — the machine-to-machine layer: MCP registries, OAuth discovery chains, capability manifests, DNS-AID, agent-readable docs.

Published as an MkDocs Material site via GitHub Pages at **https://ever-just.github.io/discoverability/**, from the public repo **ever-just/discoverability**.

## 2. The thesis

Discovery fractured. A page used to have one audience (Googlebot) and one goal (rank). Now every property has **three audiences with different mechanics**:

- **Crawlers** rank pages → you win with crawlability, structured data, and links.
- **Answer engines** cite sources → you win with answer-first content, entity clarity, real reviews, and being retrievable at answer time (Bing powers much of ChatGPT's search).
- **Agents** call endpoints → you win by being **registered, connectable, and self-describing** (registry listings, `/.well-known/*`, tool descriptions that rank for intent).

The book's core reframe, proven on customdomain.ai: **branded discovery ("find MyProduct") is easy; intent discovery ("find a solution to my problem") is the valuable, hard part** — and it must be won on all three surfaces at once.

## 3. Audience and jobs-to-be-done

Written for an operator (founder, marketer, freelancer, or an AI agent acting for them) who needs results, not theory. Every chapter must serve at least one of these jobs:

| Job | Parts that serve it |
|---|---|
| "I'm launching a site/product — make it findable from day one" | Start Here → Playbooks (SaaS launch) → Google → Bing |
| "I run a local business — get me into the map pack and AI answers" | Local Business → Google (GBP) → Playbooks (local) |
| "ChatGPT/Perplexity never mentions us" | AI Search (GEO/AEO) → Foundations (AI retrieval) → Off-site signals |
| "AI agents should be able to find and use our product" | AI Agents (MCP registry, OAuth, manifests) |
| "My site is indexed wrong / not at all" | Technical (reverse proxy, WAF, migrations) → Google (GSC) |
| "How do I know any of this is working?" | Foundations (measurement) → Playbooks (operating cadence) |

**Orientation rule:** organize by *what the reader is trying to do*, never by which internal tool or skill produced the knowledge.

## 4. Source inventory (authoritative)

The book is distilled from four source classes. When updating, mine them in this order:

### 4.1 Skills — `github.com/ever-just/agentskills` (public)

| Skill | Feeds chapters |
|---|---|
| `generative-engine-optimization` | AI Search: GEO fundamentals, off-site signals |
| `everjust-website-geo-content` | AI Search: content that gets cited (pillar/glossary/how-to clusters, answer-first copy) |
| `local-business-aeo-schema` | Local: LocalBusiness graph, FAQ schema, reviews policy; AI Search: AI crawlers, llms.txt reality check; Technical: WAF invisibility |
| `everjust-website-seo` | Google: Search Console, sitemaps; Bing: IndexNow; per-page metadata practice |
| `reverse-proxy-cms-indexing` | Technical: reverse proxies and CMS traps |
| `everjust-website-infra-views` | Technical: CMS caches; how sitewide JSON-LD actually ships |
| `everjust-tenant-domain-migration` | Technical: domain migrations |
| `mcp-server-discoverability` | AI Agents: MCP registry, OAuth discovery, tool descriptions |
| `agent-discoverability` | AI Agents: manifests, DNS-AID, registry namespace claims |
| `llm-deeplink-widget` | AI Search: Ask-AI widget |
| `marketing-site-authenticity-audit` | Local: authenticity audits; reviews policy |
| `custom-domain-email-dns-diagnosis` | Technical: domains and DNS, email trust |
| `domain-email-enumeration` | Technical: email trust (read/verification side) |
| `website-techstack-analysis` | Foundations: measurement (competitive fingerprinting) |
| `open-source-traffic-analysis` | Foundations: measurement; Case study: brogav.com |
| `web-visibility` | Cross-cutting checklist source |
| `godaddy-api` | Technical: domains and DNS |
| `white-paper-writing/ai-marketing-skills/ai-discoverability-audit` | AI Search: auditing AI visibility |
| `white-paper-writing/ai-marketing-skills/homepage-audit`, `positioning-basics` | Google: keyword strategy (messaging side) |

### 4.2 Session digests

Local Claude Code transcripts were mined (2026-08-06) into digests covering: the Heads Up on-site SEO/schema/reviews/service-area program, the customdomain.ai schema.org @graph design and AI-discoverability plan, the LLM deep-link footer build, and the everjust domain-remediation program. Re-mine transcripts under `~/.claude/projects/` when rebuilding (keyword-rank the `.jsonl` files, then extract).

### 4.3 Project memories

Distilled memory files under `~/.claude/projects/*/memory/` — richest sources: `-Users-cloudaistudio-headsup-work` (GSC API access, GitHub-org funnel, SERP tokenization crisis, live reviews), `-Users-cloudaistudio-Desktop` (customdomain schema markup, AI/agent discoverability plan).

### 4.4 Live properties (case-study ground truth)

- **customdomain.ai** — SaaS: maximal JSON-LD graph, MCP/agent discovery, GitHub org funnel, GSC baseline discipline.
- **headsupoutdoorservices.com** — local business: AEO schema, live Google reviews, service-area rebuild, authenticity remediation.
- **everjust.app tenants** (e.g. tcstartupweek.com) — multi-tenant CMS behind Host-rewriting nginx: the reverse-proxy trap catalog, domain cutovers.
- **brogav.com** — traffic-estimation methodology target.

## 5. Information architecture

Eleven parts. The nav in `mkdocs.yml` is the canonical tree; this section specifies **what each page must contain**. Global rules: every page opens with 2–4 sentences saying what the reader gets; every claim that ages carries a date ("as of 2026-07"); every how-to ends with a verification step; cross-link related pages liberally.

### Part: Start Here (`docs/index.md`, `docs/start/`)
- **index.md** — the book's pitch: three audiences, the branded-vs-intent reframe, how the book is organized, provenance (built from shipped work).
- **how-discovery-works.md** — the 2026 landscape map: which engine feeds which AI surface (Bing→ChatGPT, Google→AI Overviews), where agents look, what changed vs classic SEO.
- **choose-your-path.md** — the jobs table (§3) expanded into guided paths with links.
- **master-checklist.md** — the single unified launch checklist across all surfaces, each item linking to its chapter.
- **glossary.md** — SEO/GEO/AEO/AIO, E-E-A-T, entity, citation, crawl budget, canonical, MCP, RAG, answer engine, etc.

### Part: Foundations (`docs/foundations/`)
- **index.md** — why mechanics matter before tactics.
- **search-engines.md** — crawl → render → index → rank; crawl budget; canonicalization; how JS rendering changes things.
- **ai-retrieval.md** — the three ways LLMs know you: training data, search grounding (which engines each assistant uses), user-shared context; what a "citation" is mechanically; why answer-time retrievability ≠ ranking.
- **entities-and-trust.md** — being an entity, not just a site: consistent NAP/Organization data, sameAs graphs, E-E-A-T signals, why fabricated content is a discoverability risk (ties to authenticity audit).
- **measurement.md** — GSC baselines (record clicks/impressions/position **before** changes), AI-citation spot checks, traffic triangulation without paid tools (SimilarWeb API, Wayback CDX, Tranco, Cloudflare Radar — the brogav method), tech-stack fingerprinting of competitors.

### Part: Google (`docs/google/`)
- **index.md** — Google surface map: organic, rich results, map pack, AI Overviews — and which lever moves each.
- **search-console.md** — property setup (URL-prefix vs sc-domain), verification methods (DNS TXT, meta, service-account **API** access incl. sc-domain properties and siteOwner permission), URL Inspection API, Indexing API's real scope (JobPosting/BroadcastEvent only), reading the reports, baseline discipline.
- **sitemaps-and-robots.md** — sitemap generation/freshness (incl. CMS-generated sitemap caching gotchas), robots.txt syntax, meta robots vs robots.txt, the noindex traps catalog.
- **structured-data.md** — JSON-LD fundamentals; the `@graph` + `@id` cross-linking pattern; hand-authored vs CMS-auto blocks (never run both on the same node); deterministic generation from visible content; validation workflow (Rich Results Test + Schema Markup Validator + fetch-the-live-page).
- **rich-results.md** — which node types Google actually renders (SoftwareApplication for software; Service+OfferCatalog for pricing; FAQPage; Review stars) with the customdomain.ai dual-node model as the worked example; UnitPriceSpecification for recurring prices; eligibility policies incl. the Dec-2025 self-serving-review rule.
- **business-profile.md** — GBP setup and optimization; API access reality (quota approval lead time — apply early); reviews as the local ranking currency; connecting GBP ↔ site schema.
- **keyword-strategy.md** — the brand-name tokenization crisis (a dotted brand like "customdomain.ai" tokenizes into generic words → zero-result SERPs), head-term realism (24–36-month terms vs winnable vacant niches), intent-keyword research for GEO, the 90-day battle-plan format.

### Part: Bing & Beyond (`docs/bing/`)
- **index.md** — why Bing is disproportionately important now (feeds ChatGPT search and Copilot); the effort-to-impact case.
- **bing-webmaster-tools.md** — setup, GSC import, URL submission, diagnostics.
- **indexnow.md** — protocol, key file, per-CMS enablement (the everjust/Odoo config-param + cron pattern), which engines honor it.

### Part: AI Search — GEO/AEO (`docs/ai-search/`)
- **index.md** — GEO vs AEO vs SEO; what actually moves citations (evidence-based levers), what's snake oil.
- **geo-fundamentals.md** — the intent-vs-brand reframe; the three-surface model; ground-truth facts that kill myths; prioritized lever list.
- **content-that-gets-cited.md** — answer-first structure; pillar + glossary + how-to topical clusters; hub-and-spoke internal linking; per-page schema for content (DefinedTerm, TechArticle, FAQPage, CollectionPage); writing definitions AI quotes verbatim.
- **ai-crawlers.md** — the crawler roster (GPTBot, OAI-SearchBot, ChatGPT-User, ClaudeBot, Claude-SearchBot, PerplexityBot, Google-Extended, Bingbot); explicit robots.txt Allow-listing; verifying real access (log-level, not assumption); what each bot feeds.
- **llms-txt.md** — the honest chapter: research shows ~97% of llms.txt files get zero requests and Google says it doesn't use it; where it IS real (coding agents, docs sites); how to ship one cheaply anyway (`/docs/llms.txt` index pattern); why crawlability + schema + Bing are the real levers.
- **ask-ai-widget.md** — the "Ask AI about us" deep-link row: verified per-provider URL schemes as of 2026-07 (chatgpt.com/?q= auto-submits; perplexity.ai/search?q= auto-runs; google.com/search?udm=50&q=; claude.ai/new?q= prefills; grok.com/?q=; Gemini/Copilot broken — skip), the self-contained-prompt rule, why it's legitimate (user-initiated), fragility caveats.
- **offsite-signals.md** — the sources answer engines actually cite: review platforms, Reddit/forums, directories, comparison content, GitHub; earning presence there without astroturfing.
- **ai-visibility-audit.md** — the recurring audit: query battery across ChatGPT/Perplexity/Claude/Gemini, scoring presence/accuracy/sentiment, gap analysis, re-audit cadence.

### Part: AI Agents (`docs/agents/`)
- **index.md** — the four surfaces an agent walks from "a domain" to "a working session": registry listing → OAuth discovery → capability manifest → DNS records.
- **mcp-registry.md** — registry.modelcontextprotocol.io: server.json authoring, reverse-DNS namespace claim, DNS TXT domain verification, mirroring to mcp.so/Smithery/Glama/PulseMCP; why the registry is the single highest-leverage listing.
- **oauth-discovery.md** — the #1 silent-fail: 401 + `WWW-Authenticate` with `resource_metadata` → `/.well-known/oauth-protected-resource` (RFC 9728) → `/.well-known/oauth-authorization-server` (RFC 8414); testing the chain end-to-end.
- **manifests-and-dns.md** — `/.well-known/agent/mcp.json` capability manifests; DNS-AID (SVCB + TXT + `_index._agents` TXT + TLSA/DANE under DNSSEC) and `dns-aid verify`.
- **tool-descriptions.md** — tool descriptions as search snippets for agents: writing for agent intent ("connect a domain", "create an invoice"), annotations, naming.
- **github-as-discovery.md** — the GitHub org as a discovery funnel: GitHub search only indexes name/description/topics; docs repo as public source of truth; audience-targeted repos; awesome-lists; org profile README; AGENTS.md convention.

### Part: Local Business (`docs/local/`)
- **index.md** — the local stack: GBP + site schema + reviews + service pages + authenticity; how headsupoutdoorservices.com threads them.
- **local-business-schema.md** — the sitewide `LocalBusiness` node with `@id: #business` injected in `<head>`, resolving per-page `provider {@id}` references; per-page Service + BreadcrumbList graphs; NAP consistency.
- **faq-schema.md** — building FAQPage **deterministically from the page's visible Q&A** (extract accordion and h3/p markups) so schema always matches content per Google policy.
- **reviews.md** — the Dec-2025 policy: Review/AggregateRating must sit on a **Service** node, not LocalBusiness/Organization (self-serving ratings are ineligible and risk manual action); only real reviews (author + verbatim body); live-syncing Google Places reviews (5-review API cap; sync outside CMS sandboxes); keeping displayed ratings dynamic.
- **service-areas.md** — service-area pages that survive scrutiny: city-limits verification kills fake pages, GPS-verified photos, real per-city reviews, TIGER-derived maps; when a city page deserves to exist.
- **authenticity.md** — the authenticity audit as SEO defense: image-fetch-and-compare against captions, reused-photo dedup, fabricated-testimonial detection, over-claim checks, and the honest-generic remediation doctrine (reframe fabricated case studies as labeled generic examples).

### Part: Technical (`docs/technical/`)
- **index.md** — why infrastructure silently decides visibility.
- **domains-and-dns.md** — registrar ≠ DNS host (always `dig NS` before assuming who can write records); delegated-nameserver diagnosis; record types that matter for discovery.
- **email-trust.md** — SPF/DKIM/DMARC as a trust surface; email records are independent of site records (adding them cannot break the site); architecture options (platform mailboxes vs forwarding vs Workspace); deliverability testing (mail-tester), DMARC policy progression, BIMI.
- **reverse-proxy-cms.md** — the Host-rewrite trap catalog: CMS emits `Disallow: /` because it sees the internal host; canonicals pointing at internal hosts; the internal host itself as a crawlable duplicate (noindex it); nginx `sub_filter` canonicalization; compiled-template caches that make SEO edits invisible until restart/invalidation; pacing writes against fragile origins.
- **domain-migration.md** — cutting a live site to a new domain without losing indexing: link/canonical bases, sitewide old-domain sweep, 301-retire the old domain, keep internal machine hosts untouched, DNS cutover + certs + email-zone preservation (tcstartupweek.com cutover as the worked example), GSC change-of-address.
- **rendering-and-waf.md** — SSR vs client-rendering for crawlers/AI fetchers; the WAF invisibility trap (bot-challenge modes returning 429/challenges to Googlebot/GPTBot/PerplexityBot make a site vanish from search AND AI); how to detect (curl with bot UAs, log sampling) and fix (allow-list verified crawlers).

### Part: Playbooks (`docs/playbooks/`)
- **index.md** — how to use the playbooks; effort/impact framing.
- **saas-launch.md** — the customdomain.ai-derived sequence: GSC+Bing setup → schema graph → docs/llms.txt → GitHub funnel → MCP registry → content cluster → Ask-AI widget → measurement loop. Each step: action, owner-level effort, verification, links.
- **local-business.md** — the headsup-derived sequence: GBP → sitewide LocalBusiness graph → FAQ/service schema → real-review sync → service-area pages → authenticity pass → AI crawler allow-list.
- **ai-visibility-30min.md** — the fast audit anyone can run today: crawlability probe (curl as GPTBot/Googlebot), robots/sitemap check, schema validation, the query battery, scorecard template.
- **operating-cadence.md** — weekly/monthly/quarterly rhythm: what to check, what to log, when to re-audit, when to write new content.

### Part: Case Studies (`docs/case-studies/`)
- **index.md** — how to read the case studies; what each one proves.
- **customdomain-ai.md** — SaaS: the AI-discoverability audit → maximal schema @graph (dual SoftwareApplication+Service) → agent auth + MCP surface → GitHub org funnel → SERP tokenization crisis and the niche strategy → GSC baseline. With dates and outcomes.
- **headsup.md** — local: premium rebuild → hand-authored AEO schema → live reviews → service-area truthing → authenticity remediation → AI crawler enablement. With dates and outcomes.
- **everjust-tenants.md** — multi-tenant CMS: the reverse-proxy robots/canonical bug class, the domain-cutover method, IndexNow enablement, template-cache lessons.
- **brogav.com** — estimating a micro-traffic site's reality from open sources; confidence tiers.

### Part: Appendix (`docs/appendix/`)
- **index.md** — appendix map.
- **tools.md** — every tool referenced, grouped (Google, Bing, schema validation, DNS/email, crawl/fingerprint, traffic estimation), each with what-it's-for and a link.
- **skills-index.md** — the §4.1 table rendered for readers: chapter → skill deep links into ever-just/agentskills.
- **templates.md** — copy-paste starters: LocalBusiness @graph, SaaS dual-node @graph, FAQPage, robots.txt with AI-crawler allow-list, llms.txt, server.json skeleton, review-schema-on-Service snippet.
- **crawler-registry.md** — the user-agent table: crawler, operator, what it feeds, verification method, robots token.

## 6. Style contract

- **Answer-first.** Open every page with the takeaway. No throat-clearing, no "in today's digital landscape".
- **Operator voice.** Second person, active, concrete. "Run `dig NS example.com` first" beats "it is advisable to check nameservers".
- **Checklists and tables** for anything procedural; prose for reasoning.
- **Date-stamp volatile claims** ("as of 2026-07, Copilot's `?q=` is broken"). Policies, URL schemes, and quotas change.
- **Evidence tiers.** Distinguish "we shipped and verified this" from "documented behavior" from "community-reported". Never present a guess as a fact.
- **Show the failure.** Each chapter includes the gotchas we actually hit; anti-patterns are as valuable as patterns.
- **Cross-link** with relative links; every chapter ends with "Related" links (and its source skills where applicable).

## 7. Truth and hygiene rules (hard requirements)

1. **No secrets, ever.** No API keys, tokens, passwords, connection strings, private IPs, instance IDs, or internal file paths from the source material. Paraphrase ("a service-account key with siteOwner permission").
2. **No customer PII.** No names/emails/phones of private individuals. Public business identities (the case-study domains) are fine.
3. **Only already-public specifics.** Implementation details are fine at the pattern level; anything not already public in ever-just/agentskills stays pattern-level.
4. **Honesty doctrine applies to us too.** The book preaches authenticity — so no invented metrics, no rounded-up outcomes. If a result is unmeasured, say "unmeasured".
5. **This is a public repo.** Assume everything committed is read by competitors, customers, and LLMs (that's the point).

## 8. Build and deploy spec

- **Stack:** MkDocs + Material theme, `mkdocs.yml` at repo root, content in `docs/`, pinned via `requirements.txt` (`mkdocs-material>=9.5,<10`).
- **Theme:** tabs nav + section index pages (`navigation.indexes`), light/dark toggle, code copy, admonitions, superfences, tabbed content.
- **CI:** `.github/workflows/docs.yml` — on push to `main`: `mkdocs build --strict` → upload-pages-artifact → deploy-pages. Pages source = GitHub Actions.
- **Local check before every push:** `uvx --with mkdocs-material mkdocs build --strict` (strict mode fails on broken nav/links).
- **Nav discipline:** `mkdocs.yml` nav is exhaustive and matches `docs/` exactly; strict build enforces it.

## 9. Execution plan (for an agent running this prompt)

1. **Mine** — inventory discoverability-relevant skills in ever-just/agentskills (grep the skills tree for seo/geo/discover/llm/mcp/schema/aeo/domain/email themes); rank local session transcripts by keyword density (`search console|schema.org|llms.txt|indexnow|GPTBot|...`); read project memory dirs; fan out parallel extraction agents that produce digests. **Redact per §7 at extraction time**, not later.
2. **Design** — confirm the §5 taxonomy still fits the corpus; extend it for genuinely new themes rather than bolting misc pages on.
3. **Scaffold** — repo skeleton per §8.
4. **Write** — one writer per part, each given: this file, its part's §5 spec, the digests, and the relevant SKILL.md files. Writers produce complete chapters, not stubs.
5. **Edit** — a single editorial pass for voice consistency, cross-links, and hygiene-rule compliance (§7 scan: grep the built site for key-like strings, IPs, internal paths).
6. **Verify** — `mkdocs build --strict` locally; fix; push; confirm the Actions run deploys; spot-check the live site.
7. **Record** — update the skill index and this blueprint's §4 if sources changed.

## 10. Maintenance loop

- **New framework discovered in a work session** → capture it as a skill in agentskills first (that repo's conventions apply) → distill into the matching chapter here → add the mapping to §4.1 and `appendix/skills-index.md`.
- **Quarterly re-verification** — URL schemes (§ask-ai-widget), crawler roster, policy claims (reviews, Indexing API scope), registry/directory list. Anything that fails re-verification gets corrected in place with a dated note, not silently.
- **Case studies append, never rewrite** — add dated updates; keep the original narrative as the record.

## 11. Acceptance criteria

- `mkdocs build --strict` passes; CI deploys green; the live site renders all parts.
- Every §5 page exists with real content (no placeholders), opens answer-first, and ends with Related links.
- Every §4.1 skill is represented in at least one chapter and in the skill index.
- Zero hygiene violations (§7) in the committed tree.
- A newcomer can go from `index.md` to a concrete action plan for their situation in under 10 minutes via choose-your-path.
