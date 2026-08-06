# The operating cadence

Discoverability decays. Crawlers get re-blocked by a security default someone flipped, review counts drift away from reality, deep-link URL schemes break without notice, and a policy you built schema around gets rewritten. This is the rhythm that catches all of it: a set-once automation layer that owns the hourly loops, plus **~15 minutes a week, ~1 hour a month, and a half day a quarter** of human judgment. It's the phase every playbook in this section hands off to — [SaaS launch](saas-launch.md) Phase 9, [local business](local-business.md) Phase 9 — and the reason a movement six weeks from now will be attributable to something you did.

## At a glance

| Rhythm | Budget | What you do | Fails if you skip it |
|---|---|---|---|
| **Automated** | 0 after setup | IndexNow sweep, sitemap freshness, review sync, crawlability canaries | New pages sit undiscovered; displayed ratings drift; an outage or a WAF change goes unnoticed for weeks |
| **Weekly** | ~15 min | GSC + Bing Webmaster glance; confirm this week's content indexed; nothing new 404ing or `noindex`-ed | Silent indexing regressions run for a month instead of a week |
| **Monthly** | ~1 hr | Movement vs **baseline**; review-sync health; citation/directory drift; new competitor entrants; content-gap decisions | You optimize by vibes, and your listings quietly go stale |
| **Quarterly** | ~half day | Full AI-visibility re-audit; re-verify deep-link schemes, schema policy claims, crawler roster, robots + WAF; authenticity pass on new content | Your site accumulates dated, wrong, and broken claims — the exact thing this book tells you to fix in other people's sites |
| **On change** | minutes | After any deploy touching robots, sitemap, canonicals, domain settings, or the WAF: re-verify **on the public host** | The two catastrophes in the case studies both arrived this way |

The event-driven row is the one people drop. Both case-study disasters — a bot-challenge wall that 429'd every crawler, and a CMS domain field that `noindex`-ed every page — were *configuration changes that nobody re-verified from outside*.

---

## The automated layer (set once, runs forever)

Four jobs. Everything here is machine work; if you're doing any of it by hand monthly, you built the wrong thing.

| Job | Cadence | What it does | Proof it's alive |
|---|---|---|---|
| **IndexNow sweep** | every 6h + weekly full re-submit | Reads the **live public sitemap**, submits new/changed URLs to IndexNow (Bing, and therefore ChatGPT's retrieval path) | Cron log shows HTTP 200/202; BWT's IndexNow view lists the URLs |
| **Sitemap freshness** | every 4h (only if your CMS caches its sitemap) | Clears the cached sitemap so newly published pages appear in hours, not a half-day-plus | Publish a page, wait one cycle, `curl` the public sitemap and find it |
| **Review sync + patcher** | every 6h to daily | Pulls the live rating and review count and rewrites **every surface that repeats them** | Displayed count equals the live listing today, and still does next week |
| **Crawlability canary** | hourly or daily | Fetches homepage, `robots.txt`, and `sitemap.xml` as a browser and as bots; asserts 200, real content, no `noindex`, no `Disallow: /` | It alerts when you break something — and its silence means something because you tested it once by breaking something deliberately |

**The sweep.** Every 6 hours is a sane default: frequent enough that a publish is discovered the same day, rare enough that you never earn a 429. Make it idempotent (submit only URLs that are new or whose `lastmod` moved), standard-library only (a visibility job that dies on a dependency upgrade dies silently), and make it normalize hostnames — behind a Host-rewriting proxy, sitemaps routinely emit an internal host, and submitting those yields rejections while advertising a hostname you don't want indexed. Full pattern: [IndexNow](../bing/indexnow.md).

**No Google equivalent exists.** Say it once and stop looking: the sitemap ping endpoint died in 2023, and the Indexing API covers JobPosting/BroadcastEvent only — off-label use risks revocation with no benefit. Google freshness is a correct sitemap plus the one-time Search Console submission; after that it re-crawls forever. → [Search Console](../google/search-console.md)

**The review patcher earns its place.** In the local case study, "4.9 · 51 reviews" was hardcoded in dozens of places across the site, and within days the surfaces disagreed: 47 here, 48 there, 51 elsewhere. The fix is one stored value rendered everywhere, patched on a schedule — including the surfaces a `grep` of page content will never show you: SEO meta title/description fields stored separately by the CMS, `aria-label` attributes, and JSON-LD `reviewCount`. Run the fetch outside the CMS if its automation sandbox forbids outbound HTTP (a real constraint we hit), and never blanket find-replace a number like `4.9` — it lives inside SVG path data too. → [Reviews](../local/reviews.md)

**The canary is the cheapest insurance in this book.** Roughly:

```bash
#!/usr/bin/env bash
# canary.sh — cron hourly. Non-zero exit = a discoverability regression, page someone.
set -u; SITE="https://www.example.com"; fail=0
GB="Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
GPT="Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko); compatible; GPTBot/1.2; +https://openai.com/gptbot"

for ua in "$GB" "$GPT"; do
  code=$(curl -sS -o /tmp/body -w '%{http_code}' -A "$ua" "$SITE/")
  [ "$code" = "200" ] || { echo "FAIL status $code for UA ${ua:0:30}"; fail=1; }
  grep -qi 'name="robots"[^>]*noindex' /tmp/body && { echo "FAIL noindex on homepage"; fail=1; }
  grep -q "A DISTINCTIVE PHRASE FROM YOUR HOMEPAGE" /tmp/body || { echo "FAIL content missing"; fail=1; }
done
curl -sS "$SITE/robots.txt" | grep -qE '^\s*Disallow:\s*/\s*$' && { echo "FAIL Disallow: /"; fail=1; }
curl -sS "$SITE/sitemap.xml" | grep -qc "<loc>" || { echo "FAIL sitemap empty"; fail=1; }
exit $fail
```

Two rules for everything in this layer. **Log the status code** — a job that POSTs into the void and logs nothing is indistinguishable from a job that stopped running months ago. And **commit the job to your deploy source of truth**: on a box where deploys rsync the repo over the filesystem, a cron or nginx snippet that exists only on the box is reverted by the next deploy, and nobody notices until the sweep has been dead for a month.

---

## Weekly (about 15 minutes)

A glance, not an analysis session. Time-box it: if a check turns into work, it becomes a logged task, not a longer Friday.

- [ ] **Search Console → Pages report.** Did the indexed count move the wrong way? Any *new* exclusion reason appear? New reasons are the signal; absolute counts wobble.
- [ ] **Search Console → crawl stats.** Any spike in 5xx/429 responses, or a drop in crawl requests? A fragile origin that 503s under crawl gets throttled, and throttling looks like a ranking problem months later.
- [ ] **Did this week's new content get indexed?** Run URL Inspection on each URL you published. Expect ~days on an established domain; on a domain under three months old, expect 2–4 weeks and don't panic.
- [ ] **Bing Webmaster Tools.** Crawl information plus the IndexNow view — the only *independent* confirmation that submissions were received rather than merely sent. → [Bing Webmaster Tools](../bing/bing-webmaster-tools.md)
- [ ] **Nothing new is 404ing or `noindex`-ed.** If you deployed this week, `curl` the public `robots.txt` and one key page and read them. The canary covers the homepage; deploys break other pages.
- [ ] **Write one line in the log** — even when the line is "nothing changed". Empty weeks are what make a later spike attributable to a specific change.

What *not* to do weekly: read rank movement. Week-over-week position changes are noise, AI answers vary run-to-run for the same query on the same day, and reacting to either produces thrash. Ranking is a monthly-against-baseline question.

---

## Monthly (about an hour)

- [ ] **Movement against the baseline — never against last month alone.** Pull the same 28-day window every month and compare it to the dated baseline you recorded before any work started. Month-on-month comparisons swallow seasonality and let a slow decline read as "flat". Look at clicks, impressions, average position, *and the query mix* — a local business whose baseline was ~all brand terms is winning when commercial terms first appear in impressions, long before clicks move. → [Measurement and baselines](../foundations/measurement.md)
- [ ] **Mine striking distance.** Pull the query list; anything ranked roughly #30–70 is proven demand where the engine already considers you relevant. These are the fastest terms to move and the highest-yield hour of the month. → [Keyword strategy](../google/keyword-strategy.md)
- [ ] **Review-sync health.** Does the displayed rating and count match the live listing *today*? Check the invisible surfaces — meta descriptions, `aria-label`s, JSON-LD `reviewCount` — not just visible badges. If they've drifted, the patcher stopped running or a new page hardcoded the number again.
- [ ] **Citation and directory drift.** Spot-check the top five directory listings against your canonical NAP. Listings decay: platforms change enrollment flows, claims lapse, an aggregator re-imports old data. Because assistants pull contact facts from a narrow trusted set, a wrong number in one of them becomes a wrong number in AI answers. → [Off-site signals](../ai-search/offsite-signals.md)
- [ ] **New entrants.** Re-run three to five of your intent queries on Google and one answer engine and note *who is new*. A competitor that just published the comparison page you didn't is a content decision, not a panic.
- [ ] **Content-gap decision.** Which battery queries still return nothing of yours? Feed the answers into the improve-vs-write decision below.
- [ ] **Log the month's row** with the numbers, not adjectives.

---

## Quarterly (half a day)

This is the re-verification pass, and it exists because a specific class of thing in this field **breaks silently and stays broken**.

1. **Re-run the full AI-visibility audit.** The [30-minute audit](ai-visibility-30min.md) plus the deeper query battery, scored the same way, diffed against the previous scorecard. File the new scorecard with its date — never overwrite the old one. → [Auditing your AI visibility](../ai-search/ai-visibility-audit.md)
2. **Re-verify the LLM deep-link URL schemes.** These are undocumented, community-discovered parameters that providers change without announcement. The set verified as of 2026-07 — ChatGPT `chatgpt.com/?q=` (auto-submits), Perplexity `perplexity.ai/search?q=` (auto-runs), Google AI Mode `google.com/search?udm=50&q=`, Claude `claude.ai/new?q=` (prefills), Grok `grok.com/?q=`; Gemini and Copilot skipped as broken — is a *snapshot*, not a standard. **Verification:** click every link in your footer in a clean browser session and confirm the provider opens with the prompt intact and, where expected, actually submits. Dead links in an "Ask AI about us" row are worse than no row. → [The Ask-AI widget](../ai-search/ask-ai-widget.md)
3. **Re-check every schema policy claim against current Google documentation.** Policies in this area move fast and in both directions — self-serving `aggregateRating` on LocalBusiness/Organization became ineligible in December 2025; FAQ rich results were dropped in May 2026 (the markup still feeds Bing and AI engines); the Indexing API's sanctioned scope has stayed narrow but the enforcement language hasn't. Re-read the source docs, then fix anything your markup or your own written playbooks assert. → [Structured data](../google/structured-data.md), [Rich results](../google/rich-results.md)
4. **Refresh the AI crawler roster.** Vendors ship new tokens and split traffic three ways per vendor — on-demand fetchers, search bots, training crawlers — and a roster written six months ago is missing some. Update robots.txt, then re-fetch it from the public host. → [AI crawlers](../ai-search/ai-crawlers.md), [crawler registry](../appendix/crawler-registry.md)
5. **Re-verify robots.txt *and* the WAF still let every crawler through.** Run the bot-UA matrix again. Security vendors change defaults, and "temporary" tightening becomes permanent; this is exactly how a live business ended up with `site:` returning zero pages while the site looked perfect in a browser. Remember the honesty caveat: a spoofed-UA curl is a detector, not a proof — cross-check with `site:` and URL Inspection. → [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md)
6. **Run the authenticity pass on everything published since last quarter.** New images against their captions, new testimonials against real published reviews, new statistics against sources, new badge claims against the issuer. Drift back into fabrication happens page by page, quietly. → [Authenticity audits](../local/authenticity.md)
7. **Diff the sitemap against robots.txt**, and confirm sitemap `<loc>` entries and canonicals still carry the public host. Proxy configs and CMS upgrades reintroduce this. → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)
8. **If you have an agent surface:** confirm served == advertised. Every `/.well-known/*` URL returns valid JSON, the registry entry still resolves, and the discovery files exist **in git** — we found an entire discovery layer live in production and absent from the repo, one rebuild away from vanishing. → [MCP Registry](../agents/mcp-registry.md), [OAuth discovery](../agents/oauth-discovery.md), [Manifests and DNS-AID](../agents/manifests-and-dns.md)

---

## What to log and where

One **append-only** file, in the repo, next to the code — not a doc nobody can find. Newest entries at the bottom, ISO dates, corrections *appended* rather than edited. The point is attribution: six weeks from now, when impressions jump, the only way to know why is that the change is written down with a date.

Each entry answers five questions: **what changed, why, where it was verified, what you expect to move and when, and who owns it** (use roles, not names, if the log is public).

```markdown
## 2026-07-29 — cleared the CMS site-domain field (sitewide noindex fix)

**Change:** Cleared the CMS's configured site-domain field on the production tenant.
Every page had been emitting `<meta name="robots" content="noindex">` because the CMS
compares its configured domain against the host it *sees* — and the reverse proxy
rewrites that host, so the comparison could never match.

**Why:** URL Inspection reported every sampled URL as `Excluded by 'noindex' tag`,
with the crawler visiting daily and discarding. Live ~11 days.

**Verified at:** live rendered HTML on the public host (no robots meta), public
robots.txt (no `Disallow: /`), URL Inspection live test on 3 URLs. Not verified in
the CMS admin — that view was correct the whole time it was broken.

**Baseline recorded same day (90-day):** 24 clicks / 640 impressions / avg pos 14.5,
essentially all brand terms.

**Expect:** re-indexing over 2–4 weeks; commercial-term impressions before clicks.
**Owner:** site operator. **Re-measure:** 2026-08-26.

---

## 2026-08-26 — correction appended to the 2026-07-29 entry

Re-measured at the scheduled date. [numbers]. The original entry's "2–4 weeks"
expectation was a documented-timeline estimate, not a promise; observed [x].
Original entry left unchanged.
```

Keep four other artifacts alongside it, all dated and all append-only: the **baseline file** (the numbers before any work), the **AI-visibility scorecards** (one per quarter), the **citation log** (platform, URL, exact NAP used, status), and the **automation logs** (status codes from the sweep and the canary). Together they answer every "did that work?" question you will be asked.

---

## When to write new content vs improve what exists

Default to **improving**. A new page is a permanent maintenance obligation; an existing page with proven impressions is a lever already in your hand.

| Signal | Move |
|---|---|
| Query has impressions, position ~30–70 | **Improve** — striking distance. Answer-first rewrite of the top of the page, add a comparison table and an FAQ block. Fastest movement available |
| Query has impressions, position ~8–20 | **Improve** — title/H1 intent match, depth, internal links from your strongest pages |
| No page targets the query at all, and the live SERP is thin (a dead page, a forum answer, a stub post) | **Write new** — vacant niches are the land grab, and first-mover windows close |
| Two or more thin pages chase the same intent | **Consolidate**, then 301 the retired URLs into the survivor. We did exactly this with a set of cost-guide pages folded into a pricing page — consolidation, not loss |
| AI answers cite directories, forums, and comparison posts for your category, never you | **Off-site work**, not another on-site page. Most AI citations are third-party sources; another blog post won't fix it → [Off-site signals](../ai-search/offsite-signals.md) |
| The query is your dream head term, owned by huge domains with navigational intent | **Park it.** 24–36 months and mostly not worth it — pick the operational head term instead |
| You could generate 500 city × service pages from a template | **Don't.** That's the doorway-page class, actively penalized, and it fails an honest audit → [Service-area pages](../local/service-areas.md) |

Two hard rules. **A page you won't maintain is a liability** — it will carry stale prices, dead links, and last year's policy claims into an AI answer about you. And **never publish a page whose claims you can't evidence**, however good the keyword looks; the authenticity doctrine is a discoverability strategy, not a moral flourish.

---

## Gotchas

- **Silent automation is the default failure.** Jobs that don't log status codes appear healthy forever. And remember a `202` proves submission, not reception — only Bing Webmaster Tools shows you the other end.
- **Box-only crons and nginx edits get reverted by the next deploy.** Commit them, or discover in a quarter that your freshness loop died the day someone shipped.
- **Watching ranks weekly.** Position noise and run-to-run variance in AI answers will have you "fixing" randomness. Measure on the cadence.
- **Comparing to last month instead of the baseline.** Slow declines read as flat, and seasonal bumps read as wins.
- **Verifying on the wrong host.** We found nginx serving a stale `robots.txt` on the public domain while the internal host had the corrected one — and a schema block sitting perfectly in a CMS record on a template the homepage route never rendered. Fetch what crawlers fetch.
- **One curl proves nothing.** A transient 503 makes every check falsely fail; re-fetch before declaring a regression (and before declaring victory).
- **Personalization flatters you.** Logged-in assistants that have watched you ask about your own product will mention it. Clean sessions for every audit.
- **Quarterly items that slip become permanent.** Deep-link schemes break, crawler rosters go stale, WAF rules re-tighten, policy claims rot. Nothing on that list announces itself.
- **Blanket find-replace on a number.** Rating and count values live inside SVG path data and attribute values; a naive replacement corrupts the page while "fixing" the count.
- **Skipping the boring weeks.** The log entry that says "no changes, all checks green" is what makes the next spike attributable.

## Related

- [Launch a SaaS product](saas-launch.md) — Phase 9 is this page in miniature
- [Launch a local business](local-business.md) — Phase 9 hands off here
- [The 30-minute AI visibility audit](ai-visibility-30min.md) — the quarterly re-run
- [Playbooks overview](index.md) — how the playbooks are built, and the log-append-only rule
- [Measurement and baselines](../foundations/measurement.md) · [Auditing your AI visibility](../ai-search/ai-visibility-audit.md)
- [IndexNow](../bing/indexnow.md) · [Bing Webmaster Tools](../bing/bing-webmaster-tools.md) · [Reviews](../local/reviews.md) · [The Ask-AI widget](../ai-search/ask-ai-widget.md) · [AI crawlers](../ai-search/ai-crawlers.md)
- Source skills: [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo) · [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization) · [llm-deeplink-widget](https://github.com/ever-just/agentskills/tree/main/skills/llm-deeplink-widget) · [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema) · [marketing-site-authenticity-audit](https://github.com/ever-just/agentskills/tree/main/skills/marketing-site-authenticity-audit)
