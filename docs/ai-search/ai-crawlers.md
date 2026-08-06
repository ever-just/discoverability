# AI crawlers and crawlability

**Every major AI vendor runs two or three distinct bots — a training crawler, a search-index crawler, and an on-demand fetcher — each with its own robots.txt token. Blocking the wrong one silently removes you from AI answers; blocking none is a deliberate, reasonable default for most businesses.** This chapter gives you the roster, the explicit allow-list pattern, how to verify bots can *actually* reach you (log-level, not assumption), and the honest trade-offs of blocking.

The reason this is lever #1: an answer engine can only cite what its bots can fetch. Robots rules, WAF challenges, and client-side rendering each fail silently — the site looks perfect to humans while returning nothing usable to every crawler. We've watched a live business sit at `site:` zero for exactly this reason (measured, 2026-07 — [the WAF trap](../technical/rendering-and-waf.md)).

## The roster: who crawls, and what each bot feeds

As of 2026-07 (documented by each vendor; the [appendix registry](../appendix/crawler-registry.md) tracks changes):

| Robots token | Operator | Feeds | Class |
|---|---|---|---|
| `GPTBot` | OpenAI | Model **training** corpus | Training |
| `OAI-SearchBot` | OpenAI | ChatGPT **search index** (what ChatGPT search retrieves and cites) | Search index |
| `ChatGPT-User` | OpenAI | **Live fetch** when a user's chat needs your page right now | On-demand fetcher |
| `ClaudeBot` | Anthropic | Model **training** | Training |
| `Claude-SearchBot` | Anthropic | Claude's **search** results | Search index |
| `Claude-User` | Anthropic | **Live fetch** on a user's request | On-demand fetcher |
| `PerplexityBot` | Perplexity | Perplexity's **index** | Search index |
| `Perplexity-User` | Perplexity | **Live fetch** for a user's query | On-demand fetcher |
| `Googlebot` | Google | Google Search index → **also AI Overviews / AI Mode** | Search index |
| `Google-Extended` | Google | Opt-out token for **Gemini training/grounding** — *not* Search, *not* AI Overviews | Training toggle |
| `Bingbot` | Microsoft | Bing index → **ChatGPT search + Copilot** | Search index |
| `CCBot` | Common Crawl | Open crawl corpus many model trainers ingest | Training |
| `meta-externalagent` | Meta | Meta AI training | Training |
| `Applebot-Extended` | Apple | Opt-out token for Apple AI training | Training toggle |

Three facts worth pinning:

- **`Google-Extended` is the AI-training toggle, not a search lever.** Blocking it does not remove you from Google Search *or* from AI Overviews — those ride on `Googlebot`. Sites "blocking AI" via Google-Extended while expecting to stay in AI Overviews have it exactly right; sites blocking it hoping to *appear* less in AI answers have it wrong (documented).
- **`Bingbot` is an AI crawler now.** Bing's index feeds ChatGPT search (~87% citation overlap with Bing top-organic; external research, compiled 2026-07) and Copilot. Treat Bingbot with Googlebot-level respect.
- **The three classes fail differently.** Block a *training* bot and you're absent from future model weights (slow, diffuse effect). Block a *search-index* bot and you vanish from that assistant's citations (fast, total). Block an *on-demand fetcher* and the assistant can't even read your page when a user pastes your URL. On-demand fetchers obey robots.txt (documented per vendor policy) — which means one careless `Disallow: /` kills all three classes at once.

## The explicit allow-list pattern

A permissive robots.txt (`User-agent: *` with no disallow) already admits everyone. The explicit per-bot allow-list adds two things: it survives a restrictive `*` group you may need later, and it *documents intent* — future maintainers (and auditors) see that AI access is a decision, not an accident.

```text
# Platform plumbing stays out of every index.
User-agent: *
Disallow: /admin/
Disallow: /cart
Disallow: /search?

# AI answer engines are welcome to read and cite our content.
User-agent: GPTBot
Allow: /

User-agent: OAI-SearchBot
Allow: /

User-agent: ChatGPT-User
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Claude-SearchBot
Allow: /

User-agent: Claude-User
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Perplexity-User
Allow: /

User-agent: Google-Extended
Allow: /

Sitemap: https://example.com/sitemap.xml
```

Optionally add a `Content-Signal` line (an emerging Cloudflare-championed convention, not a standard — community-reported, 2026-07) to state the same policy machine-readably:

```text
Content-Signal: search=yes, ai-input=yes, ai-train=yes
```

Deployed verbatim (minus the placeholder paths) on headsupoutdoorservices.com and the everjust tenants (measured, 2026-07). Adapt the `*` disallows to your platform's actual plumbing paths — and **diff robots against your sitemap before shipping**: one boilerplate `Disallow: /shop` was silently hiding 14 real, sellable product pages until that diff caught it (measured, 2026-07).

## Verifying real access — log-level, not assumption

A correct robots.txt proves nothing about access. Three layers of verification, cheapest first:

### 1. Curl as each bot (minutes)

```bash
for ua in "Googlebot/2.1" "bingbot/2.0" "GPTBot/1.1" "OAI-SearchBot/1.0" \
          "ClaudeBot/1.0" "PerplexityBot/1.0"; do
  code=$(curl -s -o /dev/null -w '%{http_code}' -A "$ua" https://example.com/)
  echo "$ua -> $code"
done
# Also probe /robots.txt and /sitemap.xml the same way —
# a challenge wall that gates THOSE means crawlers can't even read your directives.
```

All 200s → the edge isn't blocking (this exact loop cleared customdomain.ai for on-site work in one minute; measured, 2026-07-15). Any 403/429, or an HTML "checking your browser" body with a 200 — go to [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md) before touching anything else. Two traps in the probe itself: some WAFs block by *client fingerprint*, not UA (Cloudflare's bot fight 403s `python-urllib` while allowing curl with a browser UA — measured, 2026-07), so probe with curl and realistic UA strings; and a challenge can pass your probe but fire under load, so pair the probe with layer 2.

### 2. Read your access logs (the ground truth)

Grep a week of server/CDN logs for the tokens above:

```bash
grep -iE "gptbot|oai-searchbot|chatgpt-user|claudebot|claude-searchbot|perplexitybot|bingbot" \
  access.log | awk '{print $1, $7, $9}' | sort | uniq -c | sort -rn | head -40
```

You're looking for three things: the bots **arrive at all**, they get **200s** (not 403/429/503 — repeated 5xx throttles crawl), and they reach **content pages**, not just robots.txt. Zero requests from a search-index bot after weeks of being "allowed" usually means the index doesn't know you exist yet — that's a [Bing Webmaster Tools](../bing/bing-webmaster-tools.md)/[IndexNow](../bing/indexnow.md) problem, not a robots problem.

### 3. Verify identity when it matters

UA strings are trivially spoofed — scrapers wear `GPTBot` as a costume. When you're making allow/deny decisions at the edge (WAF rules, rate limits), verify by **published IP ranges**, not UA: OpenAI, Anthropic, Perplexity, Google, and Microsoft each publish their crawler IP ranges (OpenAI serves per-bot JSON files, e.g. `openai.com/searchbot.json`; Google and Bing support reverse-DNS verification). Allow-list verified ranges; treat UA-only "bots" as ordinary traffic (documented, as of 2026-07).

## Should you block anything? The honest trade-offs

Blocking is legitimate — it's your content. Frame the decision per *class*, because the costs differ:

| You block… | You gain | You lose | Reasonable when |
|---|---|---|---|
| Training bots (`GPTBot`, `ClaudeBot`, `CCBot`, `meta-externalagent`) | Your content stays out of future training corpora | Baseline brand knowledge in future models — assistants know you only via live search, which is retrieval you don't control | Your content *is* the product (paywalled publishing, licensed data) |
| Search-index bots (`OAI-SearchBot`, `PerplexityBot`, `Claude-SearchBot`) | Nothing measurable for most sites | **Citations in that assistant, entirely** | You have a licensing deal or a strategic reason to be absent |
| On-demand fetchers (`ChatGPT-User`, `Claude-User`, `Perplexity-User`) | Marginal scrape protection | Users can't have "their" AI read your page — even when they paste your URL, even from your own [Ask-AI widget](ask-ai-widget.md) | Almost never |

For a business that wants customers: allow all three classes. For most local businesses and SaaS products there's no IP case for blocking even the training bots — being *known* by models is an asset (measured stance we shipped on every case-study property, 2026-07). If you do block, block precisely: named tokens, specific paths, and re-run the verification loop after — robots edits have a long history of over-matching.

!!! warning "Robots.txt is a request, not access control"
    Compliant crawlers honor it; scrapers don't. Never use robots.txt to "protect" sensitive paths (it's a public map of them). Access control is authentication's job; robots is a *visibility policy* for compliant bots.

## Gotchas

- **The WAF that challenges everyone.** Bot-challenge modes ("attack mode", "I'm under attack", "bot fight") return 429/403 challenges to Googlebot, GPTBot, and PerplexityBot alike — including on robots.txt itself. The site drops out of search *and* AI simultaneously while looking fine in a browser. Diagnose with the curl loop; fix at the edge by allow-listing verified crawlers. Full treatment: [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md).
- **Client-side rendering.** A 200 with an empty `<div id="root">` is a 200 that cites nothing. Most AI fetchers read the raw HTML; verify with `view-source`/curl, not DevTools ([same chapter](../technical/rendering-and-waf.md)).
- **Repeated 503s throttle crawlers.** A server that 503s under light sequential load teaches crawlers to back off — a real indexation risk independent of robots (measured, 2026-07).
- **The reverse-proxy robots trap.** CMSes behind Host-rewriting proxies can emit `Disallow: /` or a wrong-host `Sitemap:` line because the CMS sees the internal hostname. Check the *public* host's robots.txt output, and beware edge caches serving a stale version after you fix it (measured, 2026-07 — [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)).
- **One `*` rule nukes all classes.** `User-agent: * / Disallow: /` (a staging-site leftover, a launch checklist miss) blocks training, search, *and* on-demand fetchers at once. It's the first thing to check on any "AI never cites us" complaint.
- **Roster drift.** Vendors add and rename tokens (this table is dated 2026-07). Re-verify the roster quarterly against vendor docs — the [appendix registry](../appendix/crawler-registry.md) is this book's tracked copy.

## Related

- [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md) — when the blocker isn't robots.txt
- [Sitemaps and robots.txt](../google/sitemaps-and-robots.md) — robots syntax, meta robots, the noindex traps
- [Bing & Beyond](../bing/index.md) — getting *into* the index that feeds ChatGPT
- [AI crawler registry](../appendix/crawler-registry.md) — the maintained user-agent table
- [The 30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) — the crawlability probe in context
- Source skills: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema), [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization)
