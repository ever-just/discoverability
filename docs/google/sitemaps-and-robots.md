# Sitemaps and robots.txt

**The sitemap tells crawlers what exists; robots.txt tells them what they may fetch — and on real infrastructure, both files routinely lie.** CMSs cache sitemaps until they're stale, proxies serve different robots.txt than the CMS renders, and a single misconfigured domain field can emit `Disallow: /` sitewide without anyone noticing. The rule that survives every incident in this chapter: **always verify the file at its public URL, as a crawler would fetch it** — never trust the CMS admin view, the template source, or your last deploy log.

## Sitemaps

### What belongs in one

- **Canonical URLs only** — the public host, the canonical protocol, no redirects, no `noindex`ed pages, all returning 200. A sitemap full of URLs that redirect or carry the internal hostname actively undermines you (Google treats the sitemap as a signal of your canonicals).
- **Everything you want indexed, especially the homepage and money pages.** Obvious, and violated constantly — see the freshness traps below.
- **Honest `<lastmod>`, or none.** `<lastmod>` helps Google schedule re-crawls of changed pages. Two failure modes we've shipped or audited: a CMS emitting no `<lastmod>` at all (0 of 93 URLs — minor: it aids re-crawl scheduling, not initial indexing), and the opposite anti-pattern of stamping `lastModified: now` on every URL at every build — **fake freshness**, which teaches Google your dates mean nothing.

### How Google discovers it

1. The `Sitemap: https://example.com/sitemap.xml` line in robots.txt (absolute URL, canonical host).
2. Explicit submission in [Search Console](search-console.md) — do both.
3. That's all. The sitemap **ping endpoint is dead** (removed 2023, returns 404), and Google ignores IndexNow. After the one-time submission, Google re-crawls the sitemap on its own schedule forever — your job is keeping it *true*.

A useful habit: treat the sitemap URL count as an execution ledger. One product site's count went 93 → 101 → 117 across three content waves, resubmitted at each step; if the count doesn't move after you publish, your sitemap is stale, not your memory.

### Freshness traps (shipped-and-verified on a CMS platform)

Both of these were found live on Odoo-based tenants, but the *classes* generalize to any CMS:

1. **The sitemap is cached and stale.** Odoo renders `/sitemap.xml` once and stores it as an attachment with a ~12-hour TTL — **newly published pages don't appear until the cache clears.** Fix: a scheduled job that deletes the cached sitemap file every few hours (we run 4-hourly), and a manual cache delete as part of any domain change. If your CMS caches its sitemap (most do), find the invalidation lever *before* you need it.
2. **The homepage isn't in the sitemap at all.** The CMS's page enumerator deliberately skipped `/` (expecting the route to declare itself), and a platform addon had registered the `/` route with its sitemap flag off. Result: the single most important URL missing, silently. **Open your sitemap and check that `/` and your top pages are actually in it** — takes ten seconds, catches a whole bug class.
3. **A supplementary sitemap can't live at a non-root path** for URLs above it — the sitemaps protocol scopes a sitemap to its own directory. A file served from an attachment path cannot legitimately cover `/`. Fix the generator; don't bolt on a second file.

### Verification

```bash
curl -s https://example.com/sitemap.xml | grep -c "<loc>"          # count URLs
curl -s https://example.com/sitemap.xml | grep "<loc>" | head -20  # eyeball hosts + top pages
curl -s https://example.com/sitemap.xml | grep -c "<lastmod>"      # lastmod coverage
```

You know it's healthy when: the count matches what you believe is published, every `<loc>` uses the public canonical host, the homepage is present, and GSC's sitemap report shows the same count with a recent fetch.

## robots.txt

### Syntax that matters

```text
User-agent: *
Disallow: /my/
Disallow: /checkout

# Explicitly welcome AI-citation crawlers (see the AI Search part)
User-agent: GPTBot
Allow: /

User-agent: PerplexityBot
Allow: /

Sitemap: https://example.com/sitemap.xml
```

- Rules group under the **most specific matching `User-agent`** — a bot that matches a named group ignores the `*` group entirely, so a per-bot `Allow: /` group opts that bot out of your general Disallows. Decide per bot deliberately.
- **robots.txt controls crawling, not indexing.** A URL blocked by robots can still be *indexed* from links (as a bare URL with no snippet). Corollary: never robots-block a page you're trying to deindex — Google can't fetch it to see the `noindex`.
- Longest-match wins between `Allow` and `Disallow` within a group.
- Keep the `Sitemap:` line pointing at the canonical public host.

### The serving-layer gotcha: the file the CMS renders may not be the file the world gets

This is the most instructive robots lesson we own, hit three separate ways in production:

1. **A template condition emitted `Disallow: /` behind a reverse proxy.** The CMS's robots template disallows everything when the request host doesn't match the site's configured domain — and a Host-rewriting proxy guarantees the mismatch. The admin saw a clean robots config; the public URL served `Disallow: /`, silently blocking Googlebot and every AI fetcher. Full anatomy in [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md).
2. **The proxy cached robots.txt.** After fixing the content, the CMS host served the new file while the public domain served the stale one — and `Cache-Control: no-store` plus query-string busters did *not* defeat the proxy cache. The public file is the only one that counts; purge or bypass caching for `/robots.txt` and `/sitemap.xml` specifically.
3. **App-served root files flap during deploys.** Verification files and robots served by application routes intermittently 404 while containers roll. The robust pattern for anything at the root that must never flap: serve it from the web server/proxy layer as a static exact-match location — immune to app restarts, and it doubles as the durable fix for the template bug in (1).

**Therefore: always fetch `https://your-public-domain/robots.txt` with curl** — after every deploy, domain change, or CMS upgrade. Never conclude anything from the CMS admin screen.

### meta robots vs robots.txt vs X-Robots-Tag

| Mechanism | Controls | Scope | Notes |
|---|---|---|---|
| `robots.txt` | **Crawling** | Path patterns, per bot | Blocked pages can still be indexed URL-only |
| `<meta name="robots" content="noindex">` | **Indexing** | Single page | Google must be able to crawl the page to see it |
| `X-Robots-Tag` header | **Indexing** | Any response, set at server/proxy | The right tool for non-HTML files and for entire *hostnames* — e.g. noindexing an internal duplicate host at the proxy without redirecting it |

The internal-host case is worth spelling out: a reverse-proxied CMS's internal hostname was fully public — a complete crawlable duplicate of the site. The fix was a dedicated server block for that hostname adding `X-Robots-Tag: noindex, nofollow`, deliberately **without** a redirect, because backend links, OAuth callbacks, and webhooks on that host had to keep working.

## The noindex trap catalog

Every one of these produced (or nearly produced) invisible-in-Google in production. Symptoms first, because that's how you'll meet them:

| Trap | Symptom | Cause | Fix |
|---|---|---|---|
| **CMS domain-field noindex** | GSC: every page `Excluded by 'noindex' tag`, crawled daily | Setting the CMS's "website domain" field behind a Host-rewriting proxy makes its host-match check fail → sitewide `noindex` meta **and** `Disallow: /` robots | Clear the field, or fix Host handling at the proxy; details in [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md). Re-setting the field to fix a cosmetic og:url once re-noindexed a live site *within seconds* — this trap bites twice |
| **WAF / bot-challenge wall** | `site:` shows zero pages; humans see a normal site | Edge security mode returns 429/403 challenges to Googlebot, GPTBot, PerplexityBot — including robots.txt itself | Allow-list verified crawlers / disable challenge mode; see [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md) |
| **Robots blocking money pages** | Category/product pages never index | A boilerplate `Disallow: /shop` hid 14 real, sellable product pages; only faceted/checkout URLs should have been blocked | Diff robots.txt against the sitemap: any sitemap URL matching a Disallow is a contradiction to resolve deliberately (we found 19) |
| **Internal duplicate host** | Duplicate-content signals; internal hostname ranking instead of the brand | The machine host serves the full site publicly | `X-Robots-Tag: noindex, nofollow` on that hostname at the proxy; no redirect |
| **Draft/staging flags shipped live** | New section never appears | Pages published with "indexed" off, or staging noindex env var in prod | Audit publish + index flags as separate booleans — they are independent in most CMSs |
| **Sitemap advertising the wrong host** | GSC "Duplicate without user-selected canonical"; submissions don't take | Sitemap `<loc>` emits the internal hostname behind the proxy | Rewrite at the proxy (host substitution on the sitemap response) or fix base-URL config; then delete the cached sitemap |

The meta-lesson: **noindex states are almost never chosen — they're emitted** by an interaction between config, template logic, and infrastructure. Which is why the verification loop below is non-negotiable.

## The verification loop (run after every deploy or domain change)

```bash
D=https://example.com
curl -s $D/robots.txt                                   # 1. the PUBLIC robots — read it
curl -s $D/sitemap.xml | grep -c "<loc>"                # 2. sitemap present + plausible count
curl -s $D/sitemap.xml | grep "<loc>" | grep -v "$D"    # 3. any foreign/internal hosts? (want: none)
curl -s $D/ | grep -i '<meta name="robots"'             # 4. homepage noindex check (want: none/index)
curl -sI $D/ | grep -i x-robots-tag                     # 5. header-level noindex check (want: none)
curl -s -o /dev/null -w '%{http_code}\n' -A "Googlebot/2.1" $D/   # 6. crawler UA gets 200, not 429
```

Then confirm in GSC: URL-inspect the homepage (`indexingState` should not be `BLOCKED_BY_META_TAG`), and check the sitemap report's fetch status. Five minutes, catches every trap in the catalog.

## Gotchas

- **One curl can lie.** A transient 503 makes a healthy site look broken — re-fetch before concluding a regression, and pace verification traffic on small origins (they 503 under bursts, which is itself a crawl-budget risk worth fixing).
- **Robots edits need no restart; template-level fixes might.** On cached-template CMSs, content-field changes render immediately but view/template changes can require a worker restart to serve — "I fixed it" and "it's live" are different claims.
- **The `Sitemap:` line inherits the template's host bug** — if robots is generated, its sitemap pointer breaks the same way the sitemap's `<loc>`s do. Check both.
- **`Disallow: /` + a correct canonical is still a catastrophe.** The canonical tag being right doesn't rescue a page Google may not crawl or was told not to index.
- **Blocking a bot in robots.txt removes you from what it feeds.** Blocking OAI-SearchBot/PerplexityBot removes you from ChatGPT/Perplexity answers — make AI-crawler decisions explicitly, per bot ([the crawler roster](../ai-search/ai-crawlers.md)).
- **After a domain migration**, the cached sitemap still advertises the old/internal host until deleted — cached-sitemap deletion belongs on the [migration checklist](../technical/domain-migration.md).

## Related

- [Search Console](search-console.md) — submitting sitemaps, and reading the exclusion reports these traps produce
- [Technical: reverse proxies and CMS traps](../technical/reverse-proxy-cms.md) — the full domain-field/Host-rewrite anatomy
- [Technical: rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md) — the 429 wall
- [Technical: domain migrations](../technical/domain-migration.md) — robots/sitemap steps in a cutover
- [AI Search: AI crawlers](../ai-search/ai-crawlers.md) — the per-bot allow-list decision
- [Bing: IndexNow](../bing/indexnow.md) — the push channel Google ignores but Bing honors

Source skills: [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo) · [reverse-proxy-cms-indexing](https://github.com/ever-just/agentskills/tree/main/skills/reverse-proxy-cms-indexing) · [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema)
