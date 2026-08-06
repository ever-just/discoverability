# IndexNow

**IndexNow is a one-call protocol that tells search engines "this URL changed, come re-crawl it" instead of waiting days for them to notice.** It costs about 30 lines of code, needs no API key approval, no OAuth, and no console — just a public key file at your domain root — and it feeds Bing, which feeds ChatGPT search. It is also routinely oversold: **IndexNow accelerates discovery and crawl. It does not improve ranking, and it does not touch Google.** This chapter covers the protocol, the enablement pattern that works inside a CMS, how to verify pings are actually received, and where the ceiling is.

## How it works — three pieces

1. **A key.** Any string of 8–128 characters from `a-z`, `A-Z`, `0-9`, and `-`. You generate it; nobody issues it.
2. **A key file** at `https://yourdomain.com/<key>.txt` whose body is exactly that key. This is the entire ownership proof: an engine that receives a submission fetches the file and checks it matches.
3. **A submission** — an HTTP GET (one URL) or POST (up to 10,000 URLs) to an IndexNow endpoint.

That's the whole protocol. There is no account, no quota approval, and no rate limit worth planning around.

!!! info "The key is public by design — do not treat it as a secret"
    The verification model *requires* the key to be readable by anyone who fetches your key file. So **commit it to your repo** as a default constant (env-overridable if you want per-environment keys), serve the key file unconditionally, and stop routing it through a secret store. Shipped-and-verified practice: doing it this way removed the need for a deployment secret or a production `.env` edit entirely, which is what made "ping on every deploy" trivial to add.

## Who honors it, as of 2026-08

| Engine | Honors IndexNow | Notes |
|---|---|---|
| **Bing** | Yes | The one that matters — Bing gates [ChatGPT search](index.md#who-actually-runs-on-bing) and powers Copilot |
| **Yandex** | Yes | Co-launched the protocol with Microsoft; free Yandex coverage without touching its console |
| **Seznam** (Czech) | Yes | Free regional coverage |
| **Naver** (Korean) | Yes | Free regional coverage |
| **Google** | **No** | Never adopted it. Do not tell a client IndexNow speeds up Google |

Submitting once to the shared endpoint `api.indexnow.org` forwards to all participating engines — you don't need to hit each one. Engine-specific endpoints (`bing.com/indexnow` and friends) exist and work; posting to both the shared endpoint and Bing's directly is harmless belt-and-braces, and is what we ship.

Google's freshness loop remains what it always was: a correct sitemap plus a verified Search Console property. See [Search Console](../google/search-console.md#the-indexing-apis-real-scope) for why the tempting shortcuts there are dead ends.

## Step 1 — generate a key

```bash
# 32 hex chars, well inside the 8-128 range and the allowed alphabet
python3 -c "import secrets; print(secrets.token_hex(16))"
```

Use one key per host and keep it stable. Rotating it means re-hosting the file and updating every caller for no benefit.

## Step 2 — host the key file

The file must be reachable at your **public** domain, return HTTP 200 with the key as its entire body, and ideally serve as `text/plain`.

Serve it from the layer that owns the public host — the web server or proxy — not from an application route. This is the same lesson as the [Bing verification file that flapped during rolling deploys](bing-webmaster-tools.md#step-3-manual-verification-and-the-one-that-breaks): app routes register on boot, proxies don't care.

```nginx
# substitute your generated key for YOUR-KEY in BOTH places
location = /YOUR-KEY.txt {
    default_type text/plain;
    return 200 'YOUR-KEY';
}
```

Verify from outside before submitting anything:

```bash
curl -sSi "https://example.com/YOUR-KEY.txt"
# expect: HTTP 200, body == the key, nothing else
```

!!! warning "Fixed-filename key files need `keyLocation`"
    Some CMSs and platforms serve the key at a **fixed path** (e.g. `/platform-indexnow-key.txt`) rather than `<key>.txt`, because a route can't easily be named after a value that doesn't exist yet. That is legal — but only if every submission includes a `keyLocation` field pointing at the real file URL. Without it, engines look for `<key>.txt` at the root, don't find it, and reject the submission with 403. If you're using a platform's built-in IndexNow support, **confirm its payload includes `keyLocation`** before concluding the protocol is broken.

## Step 3 — submit URLs

**One URL (GET):**

```bash
curl -sSi "https://api.indexnow.org/indexnow?url=https://example.com/pricing&key=YOUR-KEY"
```

**Many URLs (POST):**

```bash
curl -sSi -X POST "https://api.indexnow.org/indexnow" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "host": "example.com",
    "key": "YOUR-KEY",
    "keyLocation": "https://example.com/YOUR-KEY.txt",
    "urlList": [
      "https://example.com/pricing",
      "https://example.com/guides/connect-a-domain",
      "https://example.com/glossary/dns-propagation"
    ]
  }'
```

Every URL in `urlList` must belong to `host`. Up to 10,000 URLs per request.

### Reading the response

| Code | Meaning | What to do |
|---|---|---|
| **200** | Submitted | Nothing |
| **202** | Accepted; key validation pending | Nothing — this is the normal success code for a first submission. Both of our production bulk pushes returned 202 |
| **400** | Bad request — malformed payload | Fix the JSON; check `Content-Type` |
| **403** | Key invalid — file missing or contents don't match | Fetch the key file yourself; check `keyLocation` |
| **422** | URLs don't belong to the host, or key/schema mismatch | Usually an internal hostname leaking into `urlList` |
| **429** | Too many requests | Back off; you're pinging far more often than content changes |

**Shipped numbers (2026-07):** a first bulk push of **73 URLs** to both `api.indexnow.org` and `bing.com/indexnow` returned **HTTP 202** from both; a later push of 40 URLs also returned 202. Treat 202 as success and stop worrying about it.

## Step 4 — automate it

Manual pings are a demo, not a system. Two patterns, both in production:

### Pattern A — the CMS config-parameter + opt-in cron

The pattern a platform should ship so a non-technical operator can turn IndexNow on without touching code. On the everjust.app platform (Odoo-based, multi-tenant), the visibility addon implements exactly this, and it generalizes cleanly:

| Piece | Behavior |
|---|---|
| **A config parameter holding the key** (`everjust_visibility.indexnow_key`) | The single on-switch. Setting it is a structural, confirm-gated write |
| **A key-file route** | Returns **404 until a key is set**, then serves the key. The file appearing is your first confirmation the switch flipped |
| **A one-shot submit method** | Collects the site's published URLs and POSTs them; returns the HTTP status (200/202) or a falsy value if no key is set or the POST failed |
| **A cron that ships disabled** | Calls the submit method on a schedule. **Opt-in by design** — nothing is ever submitted until an operator enables both the key and the cron |

Two design decisions worth stealing. First, **ship the cron inactive**: automatic submission on a site whose sitemap is wrong or whose pages aren't ready is worse than no submission. Second, **make the key file's existence the state indicator** — "is IndexNow on?" becomes a `curl`, answerable from outside the system by anyone.

The full enablement recipe (which parameter, which confirm gates, how to fire a one-shot submit) is in the [everjust-website-seo skill](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo).

### Pattern B — the sitemap-diffing cron

For sites whose CMS has no built-in support, a standalone job outperforms on-publish hooks because it catches *every* change, including ones made outside the publishing flow. The version running in production since 2026-07:

- Runs **every 6 hours**, plus a **weekly full re-submit** of the whole sitemap.
- Reads the **live public sitemap** (not a database query) — so it sees exactly what crawlers see.
- Submits only URLs that are **new or whose `lastmod` moved** since the last run; **idempotent**, so a double-run costs nothing.
- **Normalizes the internal hostname to the public one** before submitting. Behind a Host-rewriting proxy, sitemaps routinely emit internal hosts ([Trap 3](../technical/reverse-proxy-cms.md#trap-3-sitemap-and-canonicals-advertise-the-internal-host)) — submitting those yields 422 and, worse, advertises a hostname you don't want indexed.
- **Standard library only**, no dependencies. A visibility job that breaks on a dependency upgrade is a visibility job that silently stops running.

Also ping on deploy. A publish-time hook plus a 6-hourly sweep is belt-and-braces at near-zero cost.

## Step 5 — verify it's actually working

Four checks, cheapest first. Do all four the first time; do the last two on an ongoing basis.

1. **The key file resolves on the public host.** `curl -sSi https://example.com/<key>.txt` → 200, body is the key. If a platform serves a fixed filename, confirm submissions carry `keyLocation`.
2. **Submissions return 200/202.** Log the status code in your cron. A job that POSTs into the void and logs nothing is indistinguishable from a job that isn't running.
3. **Bing Webmaster Tools shows the submissions.** BWT has an IndexNow view listing received URLs — this is the only *independent* confirmation that the engine, not just your HTTP client, saw the ping. It is also the reason [BWT verification is a prerequisite](bing-webmaster-tools.md), not an optional extra: without it you are guessing.
4. **Crawl and index counts move.** Watch Bing's crawl information and indexed-URL counts over the following days. This is the outcome measure; the first three are plumbing checks.

**How you know it worked:** key file 200 → submissions 202 → URLs listed in BWT's IndexNow view → Bingbot crawl activity on those URLs within days. If steps 1–3 pass and step 4 doesn't move at all after a couple of weeks, the problem is not IndexNow — it's crawlability or indexability, and you should be in [rendering and WAFs](../technical/rendering-and-waf.md) or the [reverse-proxy trap catalog](../technical/reverse-proxy-cms.md).

## The honest limit: discovery, not ranking

This deserves its own section because it is the question every operator asks in week two.

**The real exchange (2026-07):** IndexNow was wired on a young SaaS domain, bulk submissions returned 202, and a week later the question came back — *"why are there still no results?"* The answer had three parts, and none of them were "IndexNow is broken":

1. **IndexNow only accelerates crawl and discovery.** It moves the engine's attention to a URL sooner. It contributes nothing to how that URL is *ranked* or whether an answer engine *cites* it. Faster indexing of a page nobody would rank is faster indexing of a page nobody will rank.
2. **The domain was inside the young-domain trust window.** New domains take roughly 2–4 weeks for first pages and 2–3 months for fuller coverage; no submission protocol shortens the trust curve.
3. **There was no way to observe reception, because Bing Webmaster Tools wasn't verified.** Every question about "did it work" was unanswerable. That gap — *running blind* — was graded the highest leverage-per-effort fix on the property.

So: ship IndexNow because it's cheap, permanent, and removes latency from your publishing loop. Then go do the work that actually earns citations — [content that gets cited](../ai-search/content-that-gets-cited.md), [off-site signals](../ai-search/offsite-signals.md), and a [keyword strategy you can win](../google/keyword-strategy.md). IndexNow is plumbing, and plumbing is worth exactly what plumbing is worth.

## Gotchas

- **Promising Google acceleration.** Google does not honor IndexNow. Say so out loud before someone else measures it.
- **Treating the key as a secret.** It's public by design. Hiding it in a secrets manager just makes the integration fragile for no security gain.
- **Fixed-filename key file with no `keyLocation`** → 403 on every submission, with a key file that looks perfectly fine when you fetch it.
- **Submitting internal hostnames.** A sitemap built behind a Host-rewriting proxy emits the internal host; submit those and you get 422s — and if they *were* accepted, you'd be advertising a hostname that should be [noindexed](../technical/reverse-proxy-cms.md#trap-4-the-internal-host-is-a-crawlable-duplicate). Normalize before submitting.
- **Enabling submission on a site that isn't ready.** IndexNow points crawlers at whatever you submit. If the sitemap is stale, pages are thin, or `noindex` is set sitewide, you are accelerating the crawl of a mess. Fix indexability first, then turn it on.
- **Pinging on a timer instead of on change.** Re-submitting an unchanged URL set every hour is how you earn a 429. Submit changes; do a full re-submit weekly at most.
- **Trusting your own logs as proof of reception.** A 202 means your request was accepted, not that anything was crawled. Confirm in BWT's IndexNow view.
- **Assuming a platform's built-in IndexNow is on.** Well-built implementations ship **off** — a key parameter *and* a cron, both explicitly enabled. Check both, then check the key file.
- **Expecting a rank change.** It accelerates discovery. That's the whole claim. Anything more is someone selling you something.

## Related

- [Bing Webmaster Tools](bing-webmaster-tools.md) — the console that lets you verify IndexNow was received at all
- [Bing & Beyond](index.md) — why the engines on the honor list are worth the integration
- [Sitemaps and robots.txt](../google/sitemaps-and-robots.md) — the URL source your submissions should agree with
- [Google Search Console](../google/search-console.md) — the Google-side channel, and why there's no equivalent push
- [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md) — where internal hostnames leak into sitemaps
- [everjust.app tenants case study](../case-studies/everjust-tenants.md) — the multi-tenant platform where the CMS pattern shipped
- [The operating cadence](../playbooks/operating-cadence.md) — where the weekly re-submit fits in a routine
- Source skills: [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo) · [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization) · [web-visibility](https://github.com/ever-just/agentskills/tree/main/skills/web-visibility)
