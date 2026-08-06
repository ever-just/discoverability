# GEO fundamentals

**Generative engine optimization is the work of becoming the source an AI assistant cites when someone asks a *category* question — not "find MyProduct" but "what's the best way to solve my problem".** The fundamentals are: reframe from brand to intent, understand the three surfaces you're optimizing at once, replace folklore with the measured ground truth, and spend effort down a prioritized lever list that starts with crawlability. This chapter is that foundation; the rest of the part is depth on each lever.

## The reframe: branded discovery is easy, intent discovery is the game

Ask ChatGPT "what is customdomain.ai" and — once the site is crawlable and indexed — it will answer. That's **branded discovery**, and it's table stakes. The valuable, hard problem is **intent discovery**: being the answer when the asker doesn't know you exist.

- *"How do I let users use their own domain on my SaaS?"*
- *"How much does snow removal cost in Shakopee?"*
- *"Best tool to automate TLS for customer domains"*

Nobody typing those queries knows your name. Winning them is what GEO is for — and it's measurably distinct work. On customdomain.ai we found the branded query literally returned zero results (Google tokenized the dotted brand into generic words — see [the tokenization crisis](../google/keyword-strategy.md)) while intent queries were winnable in weeks because their SERPs were vacant (measured, 2026-07).

The reframe has a second half. A product now has **two users pulling different levers**:

- **The human buyer**, who Googles or asks ChatGPT. Won with everything in this part: crawlability, Bing, answer-first content, schema, off-site citations.
- **The AI agent**, told "add custom domains to the app you just built." It doesn't read your blog — it consults registries and `/.well-known/` endpoints. That's the [AI Agents part](../agents/index.md), and for a developer-facing product it's the greenfield bet.

Keep the split explicit in your planning: findability (marketing surface) and agent-usability (API/MCP surface) are different codebases, different owners, one product story.

## The three-surface model

Every intent query can now resolve on three surfaces, and they share plumbing:

| Surface | Who answers | What feeds it | Your lever |
|---|---|---|---|
| **Classic search** | Google, Bing result pages | Google/Bing indexes | Crawlability, content, links — [Google](../google/index.md), [Bing](../bing/index.md) |
| **AI answers** | ChatGPT Search, Perplexity, AI Overviews/AI Mode, Claude, Copilot | Bing (ChatGPT, Copilot), Google (AI Overviews), Perplexity's own crawl, plus each vendor's on-demand fetchers | Everything in this part |
| **AI agents** | Agents acting for a user | MCP registries, `/.well-known/*`, tool descriptions, docs | [AI Agents](../agents/index.md) |

The non-obvious dependency: **answer engines sit on search indexes.** ChatGPT's search retrieval correlates ~87% with Bing's top organic results (external research, compiled 2026-07); AI Overviews draw on Google's index. So classic SEO didn't die — it became the substrate. A page that ranks nowhere on either index can still be cited only via the third route, a user pasting your URL — which you can encourage but not rely on ([the Ask-AI widget](ask-ai-widget.md)).

## Ground truth: the facts that kill the myths

Most GEO advice online is hype. These are the load-bearing facts, each dated and attributed. When a vendor pitch contradicts one of them, ask for their evidence.

1. **Crawlability is lever #1 — and it fails silently.** A WAF or bot-challenge mode can return 429/403 to every crawler while humans see a normal site. Measured on headsupoutdoorservices.com, July 2026: Vercel's Attack Challenge Mode challenged Googlebot, GPTBot, and PerplexityBot alike — including on `robots.txt` and `sitemap.xml`, so crawlers couldn't even read the crawl directives. Result: `site:` showed zero pages and the business's Facebook page outranked its own domain. Full diagnosis in [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md).
2. **Bing gates ChatGPT.** ChatGPT search runs on Bing; ~87% of its citations match Bing top-organic results (external research, compiled 2026-07). Google rank does *not* drive ChatGPT citations. Verify your site in [Bing Webmaster Tools](../bing/bing-webmaster-tools.md) and submit the sitemap there, not just in Search Console. In our 2026-07-15 portfolio scorecard, only one of seven live properties had done this (measured).
3. **llms.txt is largely decoration for AI chat.** Ahrefs' server-log study across ~137k domains found **~97% of llms.txt files get zero bot requests**; Google's Gary Illyes stated (2025-07-23) that Google does not use llms.txt for Search, AI Overviews, or AI Mode. Its one real consumer is coding agents reading docs sites. [The honest chapter](llms-txt.md).
4. **Structure gets extracted; prose gets skimmed.** HTML comparison tables are extracted **~81% vs ~23%** for the same facts as prose; FAQ-structured pages earn **~3x** more citations; **~44%** of ChatGPT citations come from the first 30% of a page (external research, compiled 2026-07). Hence the answer-first recipe in [Content that gets cited](content-that-gets-cited.md).
5. **Adding evidence lifts citations.** The Princeton GEO study (Aggarwal et al., 2023–24) measured **+40%** generative-engine visibility from adding statistics, **+40%** from citing authoritative sources, **+28%** from quotations — versus ~0% for keyword stuffing (external research).
6. **Rich schema correlates with citation.** ~61% of ChatGPT-cited pages carry rich schema vs ~25% of the baseline web; three or more schema types per page correlates with more citations (external research, compiled 2026-07).
7. **Most citations are third-party.** ~85–93% of AI citations point somewhere other than the brand's own site. Reddit is the #1 cited source (~40% overall; ~47% for Perplexity); G2/Capterra are top-cited for B2B SaaS; only ~11% of domains get cited by *both* ChatGPT and Perplexity, because the engines source differently (external research, compiled 2026-07). → [Off-site signals](offsite-signals.md)
8. **There is no force-index button.** Google's Indexing API is officially JobPosting/BroadcastEvent-only — off-label use risks revocation (documented). The sitemap-ping endpoint was removed in 2023 and now 404s (documented). Search Console's "Request Indexing" is a rate-limited hint (~10–12 URLs/day observed, unofficial). [IndexNow](../bing/indexnow.md) is real but feeds Bing/Yandex and friends — not Google.
9. **Google's own guidance names the myths.** Google's AI-search documentation explicitly lists AI-specific rewriting, fragmenting content into tiny chunks, and special "AI markup" as ineffective (documented). There is no secret markup; there is structure, evidence, and retrievability.
10. **Freshness is engine-specific.** Perplexity favors recent content and recrawls fresh pages in ~2–7 days (external research, compiled 2026-07); Google refreshes on its own sitemap-recrawl schedule; training corpora refresh on model releases. Publish ahead of seasonal demand for Perplexity; keep `<lastmod>` truthful for everyone ([Content that gets cited](content-that-gets-cited.md#freshness-and-the-recrawl-reality)).

## The prioritized lever list

Spend in this order. Each lever is cheap to *verify* before you invest in the next.

| # | Lever | Effort | Failure mode if skipped | Chapter |
|---|---|---|---|---|
| 1 | **Crawlability** — robots allows AI bots; no WAF challenge; server-rendered HTML | Minutes to check | Everything else is invisible | [AI crawlers](ai-crawlers.md) |
| 2 | **Bing pipe** — Bing Webmaster verification, sitemap submitted, IndexNow on publish | ~1 hour | ChatGPT/Copilot can't retrieve you | [Bing & Beyond](../bing/index.md) |
| 3 | **Answer-first content for intent queries** — pillar + cluster, BLUF, tables, question H2s | Days–weeks | Retrievable but never quoted | [Content that gets cited](content-that-gets-cited.md) |
| 4 | **Schema on content pages** — FAQPage, DefinedTerm, TechArticle + the site graph | Hours per page type | Weaker extraction and entity confusion | [Structured data](../google/structured-data.md) |
| 5 | **Off-site presence** — reviews, Reddit/forums, directories, comparisons, GitHub | Ongoing, slow | Capped ceiling: engines cite third parties ~85–93% of the time | [Off-site signals](offsite-signals.md) |
| 6 | **Entity clarity** — Organization + sameAs graph, consistent naming, Wikidata | Hours | Engines can't verify you exist; misattribution | [Entities and trust](../foundations/entities-and-trust.md) |
| 7 | **Measurement loop** — the query battery, citation logging, re-audit cadence | ~1 hour/month | You can't tell what worked | [AI visibility audit](ai-visibility-audit.md) |

The ordering logic: levers 1–2 gate everything and cost almost nothing to verify, so verify them *first, personally* — curl as the bots, don't assume. On the headsup engagement, the very first act was fetching the site as Googlebot/GPTBot/PerplexityBot; that one check reframed the entire program, because no amount of content work would have mattered through a 429 wall (measured, 2026-07).

## What NOT to buy

The snake-oil catalog, with reasons:

- **"Instant indexing" tools** — repackaged off-label Indexing API use (revocation risk) or dead sitemap pings. No legitimate general-purpose force-index exists (documented).
- **llms.txt packages sold as "AI SEO"** — see fact 3. Ship the cheap version for docs if you like ([decision rule](llms-txt.md#the-decision-rule)); never pay for it as a citation lever.
- **"AI-optimized rewriting" services** — Google names AI-specific rewriting and tiny-chunk fragmentation as myths (documented). Structure and evidence work; dialect-for-robots doesn't.
- **Guaranteed mentions / paid citations** — paid placement is named ineffective in Google's guidance, and astroturfed community presence gets deleted and reputationally backfires ([Off-site signals](offsite-signals.md)).
- **Fake reviews or invented aggregate ratings** — a policy violation with a manual-action risk (Google's Dec-2025 review policy — [Reviews](../local/reviews.md)), and an authenticity failure engines increasingly detect.
- **"OKF" (Google Open Knowledge Format) as an SEO play** — it's a 2026 Google Cloud spec for feeding *your own internal agents* knowledge; it is unrelated to indexing or external discovery (documented).

## Verify it yourself

Don't take this chapter's word for your own site. The 15-minute check:

```bash
# 1. Can the AI bots actually fetch you? (200 = yes; 403/429 = you're invisible)
for ua in "GPTBot" "OAI-SearchBot" "ClaudeBot" "PerplexityBot" "bingbot"; do
  echo "== $ua =="
  curl -s -o /dev/null -w '%{http_code}\n' -A "$ua" https://YOURSITE.com/
done

# 2. Does robots.txt allow them? Does the sitemap advertise the right host?
curl -s https://YOURSITE.com/robots.txt
curl -s https://YOURSITE.com/sitemap.xml | head -20

# 3. Is your content in the HTML at all, or client-rendered?
curl -s https://YOURSITE.com/ | grep -i "a distinctive phrase from your homepage"
```

Then ask ChatGPT and Perplexity three of your intent queries and note who gets cited — that's your displacement target and your baseline ([the full audit](ai-visibility-audit.md)).

## Gotchas

- **Optimizing only for Google.** The reflex of a decade of SEO. Bing gates ChatGPT and Copilot; check [Bing Webmaster Tools](../bing/bing-webmaster-tools.md) is verified before polishing another Google detail.
- **Trusting tool summaries over raw responses.** A fetch-and-summarize tool once reported missing alt text that a raw curl disproved (measured, 2026-07). Verify claims against the actual HTML an engine would see — `view-source`, not DevTools' rendered DOM.
- **Point-in-time audits rot.** Three of ten findings in one audit had already been fixed by the time implementation started (measured, 2026-07). Re-verify findings against the live site before acting on them.
- **Winning the argument, losing the entity.** All the content in the world doesn't help if engines can't resolve *who you are* — duplicate Organization nodes, inconsistent naming, and a tokenizing brand name each sabotage citation attribution. Fix the [entity layer](../foundations/entities-and-trust.md) alongside content.
- **Measuring nothing.** Record your baseline (GSC + Bing + the query battery) *before* the program starts, or you'll never prove what moved. → [Measurement](../foundations/measurement.md)

## Related

- [Content that gets cited](content-that-gets-cited.md) — the on-site execution of these fundamentals
- [AI crawlers and crawlability](ai-crawlers.md) — lever #1 in depth
- [Off-site signals](offsite-signals.md) — the highest-ceiling lever
- [How AI finds and cites](../foundations/ai-retrieval.md) — the retrieval mechanics
- [Keyword and SERP strategy](../google/keyword-strategy.md) — intent-query research and winnability triage
- Source skill: [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization)
