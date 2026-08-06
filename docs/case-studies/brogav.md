# brogav.com — estimating traffic with no paid tools

brogav.com is a micro-traffic B2B site — roughly **1,200 monthly visits** — that became the validation target for a traffic-estimation method built entirely from free, public data sources. No SimilarWeb Pro, no Ahrefs, no SEMrush. The output is a quantitative traffic and keyword profile with explicit confidence ratings, produced by triangulating five independent sources (June 2026).

!!! abstract "What this proves"
    - **You can estimate any site's traffic for free** — including competitors and prospects — if you triangulate rather than trust one source.
    - **Confidence is part of the answer.** A single estimate is a guess; three independent sources agreeing is a finding. The method makes you state which one you have.
    - **Precision degrades predictably at the bottom.** Below roughly 5,000 monthly visits, the free panel-based estimates get unreliable in *knowable* ways — and knowing that is what keeps you from over-claiming.
    - **Competitor keyword data is the real prize.** The most valuable output usually isn't the target's visit count; it's the search volumes their competitors rank for.

## The five sources

Each answers a different question, and none is trustworthy alone.

| Source | What it gives you | Where it fails |
|---|---|---|
| **SimilarWeb free API** | Monthly visits (3 months), global/country rank, bounce rate, pages/visit, traffic-source split, top keywords with volumes and CPC, top countries | Unreliable below ~5k visits/mo; may return 403/504 or empty for small sites; throttles after ~5–10 queries |
| **Wayback Machine CDX API** | Every archived capture with timestamp and status — a proxy for crawl frequency, site age, and page-count growth | Crawl frequency reflects archivist interest, not only popularity; rate-limited to roughly 15 req/min |
| **Tranco list** | Whether the domain ranks in an aggregated top-1M list | Binary at this scale — absence just means "outside the top million", which is most sites |
| **Cloudflare Radar** | DNS-based popularity bucket (top 100 / 1K / 10K / 100K / 1M) | 403 means "not ranked"; automated access can be challenged |
| **Google Trends + web search** | Relative interest over time; third-party estimates, PR mentions, industry reports | Trends is browser-only (no free programmatic API) and relative, never absolute |

## The method

**1 · Pull the panel estimate.** Query the SimilarWeb free endpoint for the target. Extract visits, rank, engagement, traffic-source split, and top keywords.

```bash
curl -s "https://data.similarweb.com/api/v1/data?domain=example.com"
```

**2 · Establish age and trajectory from the archive.** The CDX API gives first-capture date (when the site became visible to crawlers) and monthly capture volume (a rough activity/complexity trend).

```bash
curl -s "https://web.archive.org/cdx/search/cdx?url=example.com/*&output=json\
&fl=timestamp,original,statuscode,mimetype&collapse=timestamp:6&limit=500"
```

Count captures per month, count unique URLs, and read the shape: rising unique-URL counts mean the site is growing; a flat line for years means it isn't.

**3 · Bound the magnitude.** Tranco and Cloudflare Radar don't give you a number — they give you a **ceiling**. A domain absent from both is confidently outside the top million, which for a micro-traffic site is the expected, corroborating result.

**4 · Mine competitors for keyword intelligence.** This is the step people skip and shouldn't. Query the same free endpoint for three or four competitors: their keyword lists reveal what buyers in the category actually search for, with real monthly volumes and CPCs — intelligence the target site alone can never give you.

**5 · Triangulate and rate confidence.**

| Confidence | Criterion |
|---|---|
| **High** | 3+ independent sources agree on the order of magnitude |
| **Medium** | 2 sources agree, or 1 strong source with corroborating context |
| **Low** | A single source, or sources that disagree — report as a range, and say so |

## What it looked like for brogav.com

A micro-traffic B2B site at roughly **1,200 monthly visits**: present in panel data but near its reliability floor, absent from both Tranco and Cloudflare Radar (consistent, and expected at that size), with an archive history establishing site age and a modest page count. The individual signals were each weak; agreeing on the same order of magnitude is what made the conclusion usable.

**The strategic read matters more than the number.** At ~1,200 visits/month you are not going to win contested head terms — the [keyword strategy](../google/keyword-strategy.md) chapter's vacant-niche argument applies directly. Micro-traffic sites win with specific, low-competition, high-intent content, and the traffic estimate's real job is to set that expectation honestly before anyone budgets for a head-term campaign.

## The numbers

- **~1,200** monthly visits (the validated target figure, June 2026)
- **5** independent free sources triangulated
- **~5,000 visits/month** — the approximate floor below which panel-based estimates become unreliable
- **3+** agreeing sources required for a high-confidence rating
- **Unmeasured:** the true visit count. There was no analytics access to validate against — which is precisely the situation the method exists for, and precisely why confidence tiers are mandatory rather than decorative.

## What worked

- **Triangulation over authority.** No single free source is trustworthy at this scale; agreement across independent methodologies is.
- **Stating confidence explicitly.** It converts an estimate into something a reader can act on proportionally.
- **Competitor keyword mining.** Consistently the highest-value output, and free.
- **Negative results as evidence.** "Absent from Tranco" is a real, corroborating data point, not a failed query.

## What failed

- **Trusting traffic-source splits at micro scale.** Below the reliability floor, channel attribution can read 0% across the board — an artifact, not a finding.
- **Expecting precision.** These are order-of-magnitude estimates. Reporting "1,247 visits" from panel data would be false precision.
- **Rate limits.** Both the panel API and the archive throttle; batch queries need pacing.

## What we'd do differently

- Pull competitor keyword data **first** — it's the most valuable output and doesn't depend on the target resolving.
- Record the raw JSON responses with their date, since these free endpoints change shape and rate limits without notice.
- For any site you actually control, none of this is necessary: [Search Console](../google/search-console.md) gives you ground truth. This method is for sites you *don't* own.

## Chapters this case feeds

| Lesson | Chapter |
|---|---|
| Triangulate free sources; rate confidence explicitly | [Measurement and baselines](../foundations/measurement.md) |
| Which free tool answers which question | [Tool directory](../appendix/tools.md) |
| Micro-traffic sites win niches, not head terms | [Keyword and SERP strategy](../google/keyword-strategy.md) |
| Own the site? Use ground truth instead | [Google Search Console](../google/search-console.md) |

## Related

- [Measurement and baselines](../foundations/measurement.md) — where this method lives in practice
- [Keyword and SERP strategy](../google/keyword-strategy.md) — the vacant-niche argument this data supports
- [Tool directory](../appendix/tools.md) — every endpoint referenced above
- [customdomain.ai](customdomain-ai.md) — the contrasting case: a site you own, measured with real Search Console baselines
- Source skill: [open-source-traffic-analysis](https://github.com/ever-just/agentskills/tree/main/skills/open-source-traffic-analysis)
