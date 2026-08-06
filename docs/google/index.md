# Google

**Google is not one surface — it's four.** The same domain can win classic organic links, rich results, the local map pack, and AI Overviews, and each is moved by a *different* lever. Most SEO advice covers only the first. This part covers the plumbing (Search Console, sitemaps, robots), the presentation layer (structured data, rich results), the local layer (Business Profile), and the strategy layer (keywords you can actually win) — all distilled from shipped work on real properties, failures included.

## The four surfaces and what moves each

| Surface | What it looks like | The lever that moves it | Chapter |
|---|---|---|---|
| **Organic results** | The classic blue links | Crawlability + indexability + content matching intent + links. Everything else stands on this. | [Search Console](search-console.md), [Sitemaps & robots](sitemaps-and-robots.md), [Keyword strategy](keyword-strategy.md) |
| **Rich results** | Software cards, review stars, breadcrumbs, job postings | Structured data of the *specific node types Google renders*, matching visible content exactly | [Structured data](structured-data.md), [Rich results](rich-results.md) |
| **Map pack / Maps** | The 3-result local block | Google Business Profile + reviews + NAP consistency — mostly *not* your website | [Business Profile](business-profile.md), [Local Business](../local/index.md) |
| **AI Overviews / AI Mode** | The AI answer above the results | Everything above, plus answer-first content and entity clarity — AI Overviews draw on Google's index, so classic indexing gates it | [AI Search](../ai-search/index.md) |

The practical consequence: **fix surfaces in dependency order.** A page that isn't crawlable can't be indexed; a page that isn't indexed can't rank, render rich results, or feed AI Overviews. A Business Profile that isn't verified is invisible to the map pack no matter how good the website is. Diagnose bottom-up, then optimize top-down.

## The pipeline underneath

Every Google surface sits on the same four-stage pipeline (mechanics in [Foundations](../foundations/search-engines.md)):

1. **Crawl** — Googlebot fetches your URLs. Blocked here (robots.txt, WAF challenge, server errors) and nothing downstream exists. We watched a live business return HTTP 429 to Googlebot from a security toggle: `site:` showed zero pages while the site looked perfect in a browser.
2. **Index** — Google decides whether to keep the page. The silent killers live here: `noindex` meta tags emitted by CMS misconfiguration, canonical tags pointing at the wrong host, duplicate internal hostnames. One client site was crawled *daily* for weeks while every page was discarded as `Excluded by 'noindex' tag`.
3. **Rank** — position for a query. Domain authority, content quality, and intent match. This is the slow, competitive part — which is why [keyword strategy](keyword-strategy.md) is about picking fights you can win.
4. **Render** — how your result *looks*: title, snippet, favicon, rich result. Structured data and metadata live here. Cheap to fix, visible fast.

## Where to start

- **New site?** [Search Console](search-console.md) first — verify the property and record a baseline *before* touching anything. Then [sitemaps & robots](sitemaps-and-robots.md) to confirm Google can see what you think it can.
- **Site indexed but invisible?** [Keyword strategy](keyword-strategy.md) — you're probably chasing head terms at position 40+, or your brand name itself is unsearchable (yes, really: the [tokenization crisis](keyword-strategy.md#the-brand-name-tokenization-crisis)).
- **Local business?** [Business Profile](business-profile.md) before anything on-site — GBP and reviews outweigh your website for the map pack, and an unverified profile makes you invisible to local search *and* AI recommendations.
- **Want stars/cards in results?** [Structured data](structured-data.md) then [rich results](rich-results.md), in that order — the graph has to be correct before it can be eligible.

## What does NOT work on Google (as of 2026-08)

Verified in production or from Google's own statements — save yourself the detour:

| Tempting shortcut | Reality |
|---|---|
| Sitemap "ping" endpoint | **Dead.** Removed by Google in 2023; returns 404. Submit via Search Console and the robots.txt `Sitemap:` line. |
| Indexing API for normal pages | Officially **JobPosting/BroadcastEvent only**. Off-label use risks revocation with no benefit. Same for "instant indexing" plugins. |
| IndexNow | Google ignores it. It feeds [Bing and friends](../bing/indexnow.md) — worth doing, but never claim it speeds up Google. |
| llms.txt | Google has said it does not use it ([the reality check](../ai-search/llms-txt.md)). |
| "Request Indexing" automation | The GSC UI button has a ~10–12/day quota and can't be driven headlessly. One-time manual use for key pages, nothing more. |
| Exact-match domains | The EMD ranking bonus died in 2012. A generic dotted name can actually *hurt* — see [the tokenization crisis](keyword-strategy.md). |

The only durable freshness channel Google offers a normal site: a correct, current sitemap that Google re-crawls forever, discovered via Search Console and robots.txt. Boring, and the whole game.

## Evidence discipline

Every chapter in this part distinguishes **shipped-and-verified** (we did it on a live property and measured the result), **documented** (Google's published behavior), and **reported** (community/research findings). Volatile claims — policies, quotas, rendered rich-result types — carry dates, because they change. When you read this after mid-2026, re-verify anything dated before acting on it.

## Related

- [How discovery works in 2026](../start/how-discovery-works.md) — where Google sits in the three-layer map
- [Foundations: search engines](../foundations/search-engines.md) — crawl/index/rank mechanics in depth
- [Foundations: measurement](../foundations/measurement.md) — the baseline discipline every chapter here assumes
- [Bing & Beyond](../bing/index.md) — the other index, and why it gates ChatGPT
- [Case study: customdomain.ai](../case-studies/customdomain-ai.md) and [Heads Up](../case-studies/headsup.md) — the properties these lessons come from

Source skills: [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo) · [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization)
