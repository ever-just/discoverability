# The 30-minute AI visibility audit

Thirty minutes, six checks, one scorecard: can machines fetch you, does the index know you, and do AI assistants mention you? This exact probe sequence found both catastrophes in this book's case studies — a WAF challenge that had erased a business from search and AI simultaneously, and a site whose every page was silently `noindex`-ed — in the first few minutes of each engagement. Run it before any optimization work, and re-run it [quarterly](operating-cadence.md#quarterly-half-a-day).

**You need:** a terminal with `curl`, a browser, your domain, and the 3–5 phrases a customer would actually type when they don't know your name.

---

## Minutes 0–6 — Fetch the site the way bots do

Crawlers don't see what you see. Fetch your homepage with a browser user-agent as the control, then as each crawler that matters (UA strings current as of 2026 — the maintained list lives in the [crawler registry](../appendix/crawler-registry.md)):

```bash
SITE="https://www.example.com/"   # repeat for /docs or one key page if you have time

UAS=(
  "browser|Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36"
  "googlebot|Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
  "bingbot|Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)"
  "gptbot|Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko); compatible; GPTBot/1.2; +https://openai.com/gptbot"
  "oai-searchbot|Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko); compatible; OAI-SearchBot/1.0; +https://openai.com/searchbot"
  "claudebot|Mozilla/5.0 (compatible; ClaudeBot/1.0; +claudebot@anthropic.com)"
  "perplexitybot|Mozilla/5.0 (compatible; PerplexityBot/1.0; +https://perplexity.ai/perplexitybot)"
)
for ua in "${UAS[@]}"; do
  name="${ua%%|*}"; agent="${ua#*|}"
  printf "%-15s %s\n" "$name" "$(curl -sS -o /dev/null -w '%{http_code}' -A "$agent" "$SITE")"
done
```

If anything isn't a 200, look at the headers for the fingerprint:

```bash
curl -sSI -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" "$SITE" | head -15
```

| What you see | What it means |
|---|---|
| All 200, real HTML | Pass. Move on. |
| Browser 200, bot UAs 403/429 | Explicit bot blocking or UA filtering — confirm against the `site:` check and (if you have it) GSC URL Inspection's live test. |
| **Everything** non-browser 403/429 or challenge — even the browser-UA curl | A JavaScript-challenge wall. Non-rendering crawlers (Google's raw crawler, every AI fetcher) get only the checkpoint page: no title, no content, no schema. This is the [headsup case](../case-studies/headsup.md) — HTTP 429 with a `x-vercel-mitigated: challenge` header on every URL *including robots.txt*, `site:` at zero, the company's Facebook page outranking its own domain. |
| 200 but the body is an interstitial/challenge page | Status codes lie; `curl -sS "$SITE" \| head -50` and check it's *your* content. |
| Intermittent 503s | Fragile origin — re-fetch before concluding, then treat as real: crawlers hitting repeated 503s throttle or drop you. |

Also confirm your copy is server-rendered: `curl -s "$SITE" | grep -c "<distinctive phrase from your homepage>"` should be ≥ 1.

**One honesty caveat:** a spoofed-UA curl is a *detector*, not a proof. Some WAFs verify crawler IPs — they may pass real Googlebot while challenging your curl (false alarm), or an IP allow-list may pass real bots your curl can't emulate. The tie-breakers are the `site:` check (minute 11), GSC URL Inspection, and your server logs. → [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md)

## Minutes 6–11 — Read robots.txt and the sitemap

Fetch both from the **public** URL — not from your CMS admin, not from a staging host — and actually read them:

```bash
curl -sS https://www.example.com/robots.txt
curl -sS https://www.example.com/sitemap.xml | head -40
curl -sS https://www.example.com/sitemap.xml | grep -c "<loc>"
```

- [ ] No surprise `Disallow: /` (a reverse-proxied CMS can emit one on the public host while the admin view looks fine — a bug class we hit twice; → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md))
- [ ] AI crawlers not blocked — ideally explicitly allowed (→ [AI crawlers](../ai-search/ai-crawlers.md))
- [ ] A `Sitemap:` line pointing at the **public** host
- [ ] Sitemap `<loc>` URLs use the public host (an internal hostname leaking here means submissions and canonicals disagree)
- [ ] The homepage is in the sitemap; the URL count roughly matches your real page count
- [ ] No sitemap URL is robots-blocked (the two files must agree — we found a robots line hiding 14 real product pages)
- [ ] Nothing in `Disallow` that you actually sell from

→ [Sitemaps and robots.txt](../google/sitemaps-and-robots.md)

## Minutes 11–14 — The site: checks

In a browser: `site:example.com` on Google, then on Bing.

- **Zero results on an established domain** = crisis. Something in minutes 0–11 is the cause, or the domain carries a penalty — go to [Search Console](../google/search-console.md) and inspect.
- **Zero-to-few on a domain under ~3 months old** = normal; first pages take ~2–4 weeks, fuller coverage ~2–3 months.
- **A count far below your page count** = partial indexing; compare against the sitemap and check GSC's page-indexing report for the exclusion reason.
- Bing matters more than its traffic: it's the retrieval index behind ChatGPT search. If Google has you and Bing doesn't, you're invisible to a whole answer engine. → [Bing & Beyond](../bing/index.md)

Also run your exact brand name as a query. If your brand tokenizes into generic words and the SERP shows none of you, note it — that's the [tokenization crisis](../google/keyword-strategy.md), a strategy problem no quick fix solves.

## Minutes 14–18 — Rich Results Test one key page

Run your most important page (homepage, or your pillar page) through Google's Rich Results Test (`search.google.com/test/rich-results`), and the same URL through validator.schema.org.

- [ ] The page fetches successfully (RRT failing to fetch = your minute-0 problem, confirmed by Google itself)
- [ ] Expected item types detected, zero **errors** (warnings are usually optional-field suggestions)
- [ ] Detected content matches what's visibly on the page — schema that disagrees with the page is a policy problem, not a technicality
- [ ] Remember RRT shows only Google's *visual* rich-result types; Service/OfferCatalog/DefinedTerm nodes absent from its report still parse (that's what the validator run is for)

→ [Structured data](../google/structured-data.md), [Rich results](../google/rich-results.md)

## Minutes 18–28 — The 10-query battery

Build 10 queries. The mix matters — branded queries test entity recognition; intent queries test whether you're ever *the answer*:

| # | Type | Pattern |
|---|---|---|
| 1–2 | Branded | "what is {brand}", "{brand} reviews" |
| 3–5 | Problem intent | how your customer words the problem, no brand ("how do I let customers use their own domain", "lawn dethatching cost") |
| 6–7 | Category | "best {category} tools", "{category} for {segment}" |
| 8 | Comparison | "{incumbent} alternatives" |
| 9–10 | Local (if local) | "{service} near {city}", "who does {service} in {city}" |

Run each against **ChatGPT (with search), Perplexity, and Google** (AI Overviews appear on their own; AI Mode if available). Use clean sessions — logged out or temporary chats — so personalization and memory don't flatter you. For each query × engine, record three bits:

1. **Mentioned?** (your brand appears at all)
2. **Cited?** (your domain is linked as a source)
3. **Accurate?** (what it says about you is true)

And one more thing — **who gets cited instead**. Those domains (directories, Reddit threads, comparison posts, competitors) are your [off-site gap list](../ai-search/offsite-signals.md).

Pressed for time? Run queries 1–5 on ChatGPT + Perplexity only and mark the battery partial — the full 30-query grid belongs to the [recurring audit](../ai-search/ai-visibility-audit.md).

## Minutes 28–30 — Fill the scorecard

Copy, fill, and file it with the date — this is your baseline, and the thing every future re-audit diffs against:

```markdown
## AI visibility scorecard — example.com — YYYY-MM-DD
| # | Check                                        | Result        | Notes |
|---|----------------------------------------------|---------------|-------|
| 1 | Bot fetches (browser/Google/Bing/GPT/Claude/PPLX) | PASS / FAIL | status codes |
| 2 | robots.txt sane, AI crawlers allowed          | PASS / FAIL   |       |
| 3 | Sitemap public-host, agrees with robots       | PASS / FAIL   | N URLs |
| 4 | site: Google                                  | N pages       |       |
| 5 | site: Bing                                    | N pages       |       |
| 6 | Rich Results Test (key page)                  | N valid / N errors | |
| 7 | Battery: mentioned                            | n / 30        |       |
| 8 | Battery: cited                                | n / 30        |       |
| 9 | Battery: accurate when mentioned              | yes / no      |       |
| — | Who gets cited instead                        | domains…      |       |
```

## What each failure means

| Failed check | What it means | Where the fix is |
|---|---|---|
| 1 — bots challenged/blocked | A WAF/bot-protection mode is erasing you from search and AI at once | [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md) |
| 2 — `Disallow: /`, noindex, blocked AI bots | Robots/meta directive trap, often proxy-induced | [Sitemaps and robots](../google/sitemaps-and-robots.md), [Reverse proxies](../technical/reverse-proxy-cms.md) |
| 3 — wrong-host or contradictory sitemap | The site is telling crawlers two different stories | [Reverse proxies](../technical/reverse-proxy-cms.md), [Sitemaps](../google/sitemaps-and-robots.md) |
| 4/5 — site: empty or thin | Not indexed: gate problem, young domain, or never-submitted sitemap | [Search Console](../google/search-console.md), [Bing Webmaster Tools](../bing/bing-webmaster-tools.md) |
| 6 — schema errors | Machines can't parse your identity/offers | [Structured data](../google/structured-data.md), [Rich results](../google/rich-results.md) |
| 7 — never mentioned | Retrievability or content gap: you're not a source engines can lift | [GEO fundamentals](../ai-search/geo-fundamentals.md), [Content that gets cited](../ai-search/content-that-gets-cited.md) |
| 8 — mentioned, never cited | You exist as an entity but your pages aren't the retrieved evidence | [Content that gets cited](../ai-search/content-that-gets-cited.md), [AI crawlers](../ai-search/ai-crawlers.md) |
| 9 — mentioned but wrong | Entity confusion or stale third-party facts about you | [Entities and trust](../foundations/entities-and-trust.md), [Auditing AI visibility](../ai-search/ai-visibility-audit.md) |
| Competitors cited via Reddit/directories | Off-site presence gap — most AI citations are third-party sources | [Off-site signals](../ai-search/offsite-signals.md) |

## Gotchas

- **One curl proves nothing.** A transient 503 makes every check falsely fail — re-fetch before concluding a regression.
- **Status 200 with a challenge body** passes naive checks; read the body.
- **Spoofed-UA results have false positives and negatives** (IP-verified allow-lists) — cross-check with `site:`, URL Inspection, and logs before celebrating or panicking.
- **Battery answers vary run to run.** Same query, same day, different answer is normal — that's why you score 30 cells, record the date, and diff trends, not single answers.
- **Personalization flatters you.** Logged-in assistants that know you often mention your product because *you* keep asking about it. Clean sessions only.
- **Don't optimize mid-audit.** Finish the scorecard first — it's the baseline every fix gets measured against.

## Related

- [Auditing your AI visibility](../ai-search/ai-visibility-audit.md) — the full recurring version with scoring depth
- [The operating cadence](operating-cadence.md) — where the quarterly re-run lives
- [Launch a SaaS product](saas-launch.md) / [Launch a local business](local-business.md) — this audit is Phase 0–1 of both
- [Measurement and baselines](../foundations/measurement.md) — what to do with the numbers
- [AI crawler registry](../appendix/crawler-registry.md) — current UA strings and verification methods
