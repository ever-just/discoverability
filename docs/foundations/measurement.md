# Measurement and baselines

Record the numbers **before** you touch anything. A baseline you didn't capture is gone forever, and without one every subsequent argument about whether the work paid off is a matter of opinion. This chapter covers four instruments: the Search Console baseline (with the real numbers from a live launch), AI-visibility spot checks, traffic triangulation from open sources when you have no first-party access, and competitor stack fingerprinting.

The discipline underneath all four: **every number gets a source, a date, and a confidence level.** Estimates are marked with `~` and the method that produced them. Guesses are not published.

## 1. The Search Console baseline

Before the first fix, pull 90 days of Search Console performance data and write down five things:

| Metric | Why it's the baseline |
|---|---|
| **Clicks** | The outcome. Usually tiny and honest at the start. |
| **Impressions** | Eligibility. Proves you're indexed and being *considered*. |
| **Average position** | Distance from the money. |
| **CTR** | Separates "wrong position" from "wrong snippet". |
| **Top queries, with position** | The diagnosis. Which intents you appear for, and how far away. |

Also capture, at the same moment: indexed page count, sitemap URL count, whether the sitemap has ever actually been submitted, the `robots.txt` and `noindex` state of the public host, a JSON-LD `@type` census, and AI-crawler status codes ([the probe](ai-retrieval.md#indexed-is-not-retrievable)). Those five minutes of capture are what turn later work into evidence.

### The real customdomain.ai baseline

Pulled 2026-07-15, trailing 90 days, before any of the program's work landed:

> **~5 clicks · ~159 impressions · average position ~42**

That single line did more analytical work than any other number in the program, because of what it *ruled out*:

- **Not blocked.** 159 impressions means Google is indexing and serving the site. A crawlability problem would show near-zero impressions.
- **Not penalized.** Impressions were landing on exactly the right intent queries — the category head term around position 72, a related solution term around 45, a problem-phrased query around 58. A penalized site doesn't get shown for its category at all.
- **Not yet earned.** Position 42 average is "indexed and invisible": a young, low-authority domain being told, accurately, that it hasn't earned those terms. The strategic response wasn't more schema. It was picking queries the site could actually win.

Adjacent discovery in the same pull: the site's 101-URL sitemap **had never been submitted**. Baselines find that class of bug for free.

The local-business equivalent, pulled 2026-07-29 on headsupoutdoorservices.com just before the noindex fix: **24 clicks / 640 impressions / 3.75% CTR / average position 14.5** — and reading the query table showed it was almost entirely the brand term at position 1.8, with commercial terms nowhere (a core service query around 27, a city-plus-service query around 45). Position 14.5 looked healthy in aggregate and was entirely brand. **Always read the query breakdown; the average is a liar.**

### Reading a baseline

| Pattern | Diagnosis | Where to go |
|---|---|---|
| Near-zero impressions | Not indexed, or blocked | [Search engines](search-engines.md), [Rendering and WAFs](../technical/rendering-and-waf.md) |
| Impressions, position 30+ on the right intents | Indexed, unranked, healthy. Young domain. | [Keyword strategy](../google/keyword-strategy.md) — pick winnable niches |
| Impressions, position < 15, near-zero CTR | Snippet or intent mismatch | Titles, descriptions, [rich results](../google/rich-results.md) |
| Healthy position but 100% brand queries | No intent discovery at all | [Content that gets cited](../ai-search/content-that-gets-cited.md) |
| Impressions collapse on a known date | Something shipped. Correlate with your change log. | [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md) |

### The diagnostic that outranks the report

Search Console's **URL Inspection** tells you the exact stage a URL died at, per URL. Two findings it handed us that no aggregate report would have:

- Every page on one site: `coverageState: "Excluded by 'noindex' tag"`, `indexingState: BLOCKED_BY_META_TAG`, with a last-crawl date of *yesterday*. Google was crawling daily and discarding everything.
- A set of job pages: indexed, but rich results failing on a missing required field — an error invisible from the performance report.

Search Console is fully drivable from a service account holding siteOwner permission on a domain property, which makes baseline capture scriptable. → [Google Search Console](../google/search-console.md)

### Re-measurement cadence

| When | What |
|---|---|
| 2–4 weeks after an indexing fix | Re-pull the same 90-day window. Indexing should climb once a block is gone; nothing else moves that fast. |
| Monthly | Query mining — anything sitting at positions 11–20 is a striking-distance opportunity |
| Quarterly | Re-verify volatile claims: crawler roster, URL schemes, policy statements, directory lists |

Write the delta down next to the baseline. A measurement you didn't record is a measurement you'll re-argue.

## 2. Measure the thing that matters when analytics is absent

Not every property has analytics, and installing it is a decision (property setup, consent posture) that can stall for weeks. Don't let that block measurement — instrument the outcome instead of the traffic.

On a local-business site with **no analytics installed anywhere**, we tagged the lead-capture path from city pages with a distinct source value so city-page leads became attributable inside the CRM. It measures the thing that matters — leads — with no third-party dependency and no consent question. The finding it surfaced: **0 of 242 CRM leads were attributable to the service-area section**, which reframed a "should we expand city pages?" conversation entirely.

Related instrumentation gap worth checking on any funnel: if your flow has branches (a fast path and a fallback), persist which branch each attempt took and its outcome. We found a connection flow where success-versus-fallback rates were simply unmeasurable because neither was recorded.

## 3. AI-visibility spot checks

Search Console cannot see AI answers. The instrument is a **query battery you run yourself**, on a schedule, logging results.

The minimum viable version:

1. Write 10–20 queries a real buyer would ask — problem-aware, solution-aware, and vendor-aware. Use literal phrasings, not keyword-ese.
2. Run each through ChatGPT, Perplexity, Google AI Mode, and Claude.
3. Log, per run: date, engine, query, whether you appear, **which domains were cited**, and whether what was said about you is accurate.
4. Re-run monthly. Score movement, not a single reading.

Two mechanical warnings. First, answer engines are **non-deterministic** — the same prompt yields different citations across runs, accounts, and sessions. One appearance is not a win and one absence is not a loss; you're tracking a rate. Second, the cited-domain column is the valuable one: it tells you which third-party sources the engine trusts for your category, which is your [off-site](../ai-search/offsite-signals.md) target list.

The one first-party AI instrument that exists: **Bing Webmaster Tools' AI performance reporting** shows Copilot-side citation data — and it only works if the property is verified there. On a portfolio audit we graded a property's AI-visibility observability an **F** for exactly this reason: the operator was "running blind," and the fix was a ten-minute verification. → [Bing Webmaster Tools](../bing/bing-webmaster-tools.md)

The full protocol — query taxonomy, scoring rubric, gap analysis, re-audit cadence — is [Auditing your AI visibility](../ai-search/ai-visibility-audit.md). The fast version is the [30-minute AI visibility audit](../playbooks/ai-visibility-30min.md).

## 4. Traffic triangulation without paid tools

For any site you don't own — a competitor, an acquisition target, a prospect — you can build a defensible traffic picture from free sources. This is the method validated on brogav.com in June 2026. **The rule is 3+ independent sources per claim, each with a confidence tier.**

| Source | Endpoint | What you get | Limits |
|---|---|---|---|
| SimilarWeb free data endpoint | `data.similarweb.com/api/v1/data?domain=…` | ~3 months of visits, global/country rank, bounce, pages/visit, traffic-source split, top keywords with volume and CPC | Throttles after a handful of queries; 403/504 on some domains |
| Wayback CDX API | `web.archive.org/cdx/search/cdx?url=…/*&output=json` | Every archived capture: first-seen date, URL inventory, capture frequency, content types | ~15 req/min; use `collapse=timestamp:6` for monthly granularity |
| Tranco | `tranco-list.eu/api/ranks/domain/…` | Rank in an aggregated top-1M list | Empty result = outside top 1M |
| Cloudflare Radar | `radar.cloudflare.com/api/v1/ranking/domain/…` | DNS-based ranking bucket | 403 = not ranked; may challenge automation |
| Chrome UX Report | BigQuery | Presence implies roughly 10k+ monthly pageviews | Needs BigQuery access |

```bash
D=example.com
curl -s "https://data.similarweb.com/api/v1/data?domain=$D" | python3 -m json.tool | head -40
curl -s "https://tranco-list.eu/api/ranks/domain/$D"
curl -s "https://web.archive.org/cdx/search/cdx?url=$D/*&output=json&fl=timestamp,original,statuscode,mimetype&collapse=timestamp:6&limit=500" \
  | python3 -c "
import json,sys
from collections import Counter
rows=json.load(sys.stdin)[1:]
print('captures', len(rows), '| unique URLs', len({r[1] for r in rows}))
print('first seen', min(r[0] for r in rows)[:6])
print('by month', dict(sorted(Counter(r[0][:6] for r in rows).items())))"
```

### Confidence tiers

Publish every metric with the number of independent sources that agree:

| Metric | Value | Sources agreeing | Confidence |
|---|---|---|---|
| Global rank | outside top 1M | SimilarWeb + Tranco + Cloudflare Radar (3) | **HIGH** |
| Site age / first indexed | first Wayback capture | Wayback (1), corroborated by cert history | HIGH |
| Page-count trajectory | URL inventory over time | Wayback (1) | HIGH |
| Monthly visits | `~1,200` at peak | SimilarWeb (1) | **LOW–MEDIUM** |
| Traffic-source split | 0% across all channels | SimilarWeb (1), below its reliability threshold | LOW |

The brogav.com result, as an example of what a defensible answer looks like: **~1,246 visits/month at peak (May 2026), 408 archived captures, 130 unique URLs, first capture June 2023, not in Tranco or Cloudflare Radar rankings.** Conclusion: a micro-traffic site functioning as a digital brochure rather than a lead engine, with a social profile that outweighs its web presence. Note how the *high-confidence* claims (rank, age, page growth) carry the conclusion, and the low-confidence visit estimate is bounded with `~`. → [brogav.com case study](../case-studies/brogav.md)

**Three caveats to state every time:**

1. Below ~5,000 monthly visits, SimilarWeb estimates are directional at best and the traffic-source split can read 0% across every channel.
2. Wayback capture frequency is a **proxy** influenced by the Archive's own prioritization, not a traffic measurement.
3. Three sources confirming an *absence* ("not ranked anywhere") is stronger evidence than one source estimating a presence.

**The competitor-keyword trick:** query the free SimilarWeb endpoint for competitors rather than only the target. Their keyword rows return real monthly volumes and CPCs for your category — demand data you can't get from the target domain alone, and the cheapest input to [keyword strategy](../google/keyword-strategy.md) there is.

## 5. Competitor stack fingerprinting

Ten minutes of probing tells you what a competitor has actually built, which is a better prioritization input than looking at their homepage.

```bash
D=competitor.com

# Platform, CDN, server, frameworks
httpx -u "https://$D" -tech-detect -status-code -title -server -cdn -json -follow-redirects

# Infrastructure and email posture
dig +short NS "$D"; dig +short MX "$D"; dig +short TXT "$D"; dig +short TXT "_dmarc.$D"

# Every subdomain they've ever issued a certificate for
curl -s "https://crt.sh/?q=%25.$D&output=json" | python3 -c "
import json,sys
print('\n'.join(sorted({n.strip().lstrip('*.') for e in json.load(sys.stdin)
  for n in e.get('name_value','').split('\n') if n.strip()})))"

# Discoverability posture
curl -s "https://$D/robots.txt"
curl -s "https://$D/sitemap.xml" | grep -c '<loc>'
curl -s "https://$D/" | grep -o '"@type": *"[A-Za-z]*"' | sort | uniq -c
curl -s -o /dev/null -w '%{http_code}\n' "https://$D/llms.txt"
```

What each line buys you, read as discoverability intelligence:

| Signal | What it tells you |
|---|---|
| Sitemap `<loc>` count | The real size of their content program — and whether a "5,900-page" competitor is a content machine or a thin programmatic farm |
| JSON-LD `@type` census | Whether they've done entity work at all. Most haven't. |
| `robots.txt` AI-bot rules | Whether they're courting or blocking answer engines |
| CDN / WAF | Whether they risk the [bot-challenge invisibility](../technical/rendering-and-waf.md) trap you just checked yourself for |
| Subdomains from certificate transparency | Docs hosts, API hosts, status pages — the surfaces they're actually investing in |
| MX / SPF / DMARC | Email maturity, and a proxy for operational rigor → [Email trust](../technical/email-trust.md) |

Absences are findings. A competitor with 5,000 indexed pages and no structured data is beatable on entity clarity; one with a lean site and a deep third-party footprint is telling you the category is won off-site.

## Honesty rules that keep measurement usable

Learned the hard way, mostly by being wrong first:

- **Every number needs a real source.** A keyword analysis of ours was rejected and rebuilt from scratch because it presented modelled numbers as measured ones. Mark estimates with `~` and name the method.
- **Re-fetch before declaring a regression.** A single `curl` can hit a transient 503 and make every check falsely "miss". Small origins do this under light concurrency. Pace verification sweeps sequentially with backoff.
- **Grep the rendered page, not the stored template.** Content lives in fields your template grep will never see — meta-description fields, `aria-label`s, compiled assets.
- **Live is the source of truth.** When multiple people or tools edit one site, the served bytes outrank local files, database rows, and anyone's memory of what shipped.
- **Verify by fetching, not by attribution.** "The other workstream said they did it" is not evidence. Fetch it.
- **Run an adversarial pass before production changes.** A review pass whose explicit mandate is to *refute*, not confirm, has repeatedly returned needs-changes on work that looked finished — including on a headline fix that didn't address the problem it was written for.

## Gotchas

1. **No baseline.** The most common and least recoverable error. Capture before you fix.
2. **Reading the average position and stopping.** A 14.5 average that's 100% brand terms is not a healthy site. Read the query table.
3. **Expecting movement in days.** Indexing changes show in 2–4 weeks; ranking and citations take months. Judging a fix at 72 hours produces false negatives and panic reversals.
4. **Treating a single AI answer as a measurement.** Non-deterministic. Score a battery repeatedly.
5. **Publishing a SimilarWeb number for a micro-traffic site as fact.** Below ~5k/month the error bars swallow the estimate. Tier it LOW and lead with what three sources agree on.
6. **Confusing Wayback crawl frequency with traffic.** It's a proxy for site activity, mediated by the Archive's own scheduling.
7. **Measuring pageviews when the business runs on leads.** Instrument the outcome; a source tag in a CRM beats an analytics install you can't get approved.
8. **Trusting a green build, a tool summary, or a dashboard over the served page.** CI can log success and land nothing. Verify the artifact.

## Related

- [Search engines — crawl, index, rank](search-engines.md) — the stages a baseline diagnoses
- [How AI finds and cites](ai-retrieval.md) — what the AI-visibility battery is measuring
- [Entities, E-E-A-T, and trust](entities-and-trust.md) — the consistency work measurement keeps honest
- [Google Search Console](../google/search-console.md) — property setup, API access, and the reports
- [Auditing your AI visibility](../ai-search/ai-visibility-audit.md) — the full query-battery protocol
- [The operating cadence](../playbooks/operating-cadence.md) — when to run each of these
- [brogav.com case study](../case-studies/brogav.md) — the triangulation method end to end
- [Tool directory](../appendix/tools.md) — every endpoint referenced here
- Source skills: [open-source-traffic-analysis](https://github.com/ever-just/agentskills/tree/main/skills/open-source-traffic-analysis), [website-techstack-analysis](https://github.com/ever-just/agentskills/tree/main/skills/website-techstack-analysis), [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo)
