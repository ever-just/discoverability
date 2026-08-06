# Rendering, WAFs, and bot challenges

**A WAF in bot-challenge mode can erase your site from Google and every AI assistant simultaneously, while humans see nothing wrong at all.** We watched it happen: a business's domain returned HTTP 429 challenge pages to every crawler, `site:` showed **zero indexed pages**, and the company's Facebook page outranked its own domain — while the site worked perfectly in every browser. The other, quieter version of the same failure is client-side rendering: the server sends an empty shell and the content only exists after JavaScript runs, which most AI fetchers never do.

Both failures share a property that makes them lethal: **no dashboard shows them.** You find them by fetching your site the way crawlers do.

## Failure 1 — the WAF invisibility trap

### What happened (shipped incident, 2026-07)

A site on a managed platform had its "attack challenge mode" enabled. Probing it as crawlers revealed:

- Requests as `Googlebot/2.1`, `GPTBot/1.1`, `PerplexityBot/1.0`, *and* a real-Chrome UA all returned **HTTP 429** with the header `x-vercel-mitigated: challenge` and a ~34 KB JavaScript challenge page — a blanket challenge with no crawler allowlist.
- `/robots.txt` and `/sitemap.xml` were gated too — crawlers couldn't even read the crawl directives.
- `site:thedomain.com` → zero results; only a stale pre-hardening homepage title survived in the index; the brand's Facebook page outranked the owned domain for its own name.

The mechanics: a challenge requires JavaScript execution to pass. Non-rendering fetchers — most AI crawlers and search crawlers on the initial fetch — receive only the challenge HTML: no title, no content, no schema. To them, your site *is* the checkpoint page. Search presence decays to nothing; AI answer engines that ground on live retrieval ([how AI finds you](../foundations/ai-retrieval.md)) can never cite you, because retrieval fails at the fetch.

The fix was configuration, not content — a platform toggle plus crawler allowances. "The #1 SEO lever is a decision, not code."

### Detect it in five minutes

Run the battery against your **public domain** (see also the [30-minute audit](../playbooks/ai-visibility-30min.md)):

```bash
for ua in \
  "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  "Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)" \
  "GPTBot/1.1" \
  "PerplexityBot/1.0" \
  "Mozilla/5.0 (compatible; ClaudeBot/1.0; +claudebot@anthropic.com)"; do
  printf '%-12s ' "$(echo "$ua" | grep -oE 'Googlebot|bingbot|GPTBot|PerplexityBot|ClaudeBot')"
  curl -sS -o /dev/null -w '%{http_code}\n' -A "$ua" https://yourdomain.com/
done
curl -sS -o /dev/null -w 'robots: %{http_code}\n'  -A "GPTBot/1.1" https://yourdomain.com/robots.txt
curl -sS -o /dev/null -w 'sitemap: %{http_code}\n' -A "GPTBot/1.1" https://yourdomain.com/sitemap.xml
```

Reading the results:

| Signal | Meaning |
|---|---|
| 200 everywhere, real HTML | Healthy. Diff the *content* per UA anyway (some WAFs serve different bodies at 200). |
| 429/403 + `x-vercel-mitigated: challenge` | Vercel challenge mode (the incident fingerprint; also sets `cache-control: private, no-store`) |
| 403/503 + `cf-mitigated: challenge` or a "Just a moment…" body | Cloudflare challenge (Under Attack mode / bot rules) |
| 429 with `Retry-After` | Rate limiting — crawlers back off and slow-crawl you |
| 200 to browsers, 4xx to bot UAs only | A deliberate (or vendor-default) bot block — check your WAF's bot-management rules |

Two caveats that keep the test honest. First, your curl is a *spoofed* UA: a well-configured WAF may block fake-Googlebot (you) while allowing verified Googlebot — so a 4xx here is a strong warning, not final proof. Confirm with ground truth: **GSC's URL Inspection live test** (fetches with real Googlebot), GSC **Crawl Stats** (look for spikes in 4xx/5xx fetch responses), and a sample of your access logs filtered to crawler UAs and their published IP ranges. Second, a blanket challenge like the incident's returns 4xx to *everything* including real browsers' first uncookied request — which is why `site:` checks and log sampling belong in the battery.

### Fix it: allow verified crawlers, verified properly

1. **Turn off blanket challenge modes** ("I'm under attack" / "attack challenge mode") once the attack passes. They are emergency brakes, not resting states — every day enabled is a day of index decay.
2. **Allow-list crawlers by verification, not by User-Agent string.** UA strings are freely spoofable; scrapers claim to be Googlebot constantly. Every major operator publishes a real verification method (as of 2026-08):
   - **Googlebot / Bingbot**: forward-confirmed reverse DNS — the IP resolves to `*.googlebot.com`/`*.google.com` (or `*.search.msn.com`), and that hostname resolves back to the same IP. Both also publish official IP-range JSON files.
     ```bash
     host 66.249.66.1                      # → crawl-66-249-66-1.googlebot.com
     host crawl-66-249-66-1.googlebot.com  # → 66.249.66.1  (forward-confirmed)
     ```
   - **GPTBot / OAI-SearchBot / ChatGPT-User (OpenAI), ClaudeBot (Anthropic), PerplexityBot (Perplexity)**: published IP-range lists on each operator's site.
   - Managed WAFs (Cloudflare "verified bots", Vercel, AWS WAF bot control) maintain these lists for you — prefer their *verified-bot allow* categories over hand-rolled UA rules.
3. **Keep `/robots.txt` and `/sitemap.xml` outside every challenge/block rule.** Directives nobody can fetch protect nothing — and blocking robots.txt makes crawlers assume, not obey.
4. **Re-run the detection battery after the change** and confirm in GSC that live-test fetches succeed. Then request reindexing of key pages; recovery begins at the next crawl.

The roster of AI crawlers worth admitting — and what each one feeds — lives in [AI crawlers](../ai-search/ai-crawlers.md) and the [crawler registry](../appendix/crawler-registry.md).

## Failure 2 — client-side rendering

If your HTML arrives empty and JavaScript builds the page, here is who sees what (documented behavior, as of 2026-08):

- **Googlebot** renders JavaScript — but in a second wave, with delays and a crawl-budget cost; JS-dependent content and links are indexed later and less reliably than served HTML.
- **Most AI fetchers do not render.** GPTBot, ClaudeBot, PerplexityBot and answer-time retrieval fetchers overwhelmingly consume the served HTML. A client-rendered page is an empty page to the systems this book is about.

### The view-source test

```bash
curl -sS https://yourdomain.com/your-key-page | grep -c "a phrase from the page's visible copy"
```

Or in a browser: **View Source** (not Inspect — DevTools shows the post-JS DOM). If your copy, title, meta tags, and [JSON-LD](../google/structured-data.md) aren't in the raw response, they don't exist for non-rendering fetchers. Fixes, in order of preference: server-side render or statically generate the marketing/content pages (modern frameworks make this a build setting); prerender for bots as a stopgap; at minimum, ship title/meta/canonical/schema server-side even if widgets hydrate client-side.

## Failure 3 — the quiet throttlers

Not blocks, but drags with the same direction of travel (all observed in production):

- **5xx under crawl load.** A small origin that 503s under a burst teaches crawlers to slow down; repeated 503s can get pages dropped. If your own parallel probes can 503 the site, so can a crawl wave — fix origin headroom, and pace your own verification sweeps sequentially.
- **Interstitials and consent walls** that gate content behind a click perform like soft challenges for non-rendering fetchers.
- **Geo/IP blocks** on cloud ranges: many AI fetchers egress from cloud IPs — a datacenter-IP block rule quietly bans them.

## Re-test cadence

The WAF trap's defining feature is that it comes *back* — a security incident, a platform default change, or a well-meaning teammate re-enables the wall, and nothing visible breaks. Treat crawlability as a monitored invariant, not a one-time fix:

| When | What |
|---|---|
| After **any** security/WAF/CDN setting change | The full UA battery + robots/sitemap fetch |
| After platform migrations or plan changes | Same — defaults differ per plan and platform |
| Monthly (scheduled) | UA battery + `site:` sanity check + GSC Crawl Stats review ([operating cadence](../playbooks/operating-cadence.md)) |
| During any traffic anomaly investigation | Crawl-response codes in GSC before content theories |

Wire the battery into the [30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) and it stops being a separate chore.

## Gotchas

1. **"The site works fine" is testimony from the wrong witness.** Humans with JS and cookies pass challenges automatically. Only bot-eyed probes and logs tell the truth.
2. **Challenge modes left on after the attack.** The emergency brake became the parking brake; the index quietly drained. Calendar the review when you enable one.
3. **Allow-listing by User-Agent string.** You'll admit every scraper that lies and still trip real crawlers coming from unexpected IPs. Verify by rDNS/IP ranges — or use your WAF's verified-bot category.
4. **Blocking robots.txt itself.** Inside a challenge rule, your directives are unreadable; crawlers can't even learn what you allow.
5. **A spoofed-UA 200 proving less than you think.** Some WAFs allow *fake* Googlebot (no rDNS check) while a misconfigured rule blocks *real* Googlebot. Cross-check with GSC live test and logs, not curl alone.
6. **Client-rendered schema.** JSON-LD injected by JavaScript is invisible to non-rendering fetchers — exactly the audiences schema exists for. Serve it in HTML.
7. **Rate-limit rules sized for humans.** A crawl wave from one operator's range can look like an attack to a per-IP throttle; verified crawlers deserve wider limits.
8. **Testing the staging/internal host.** Its WAF posture differs from the public domain's. Probe what crawlers fetch ([the meta-lesson](index.md)).

## Related

- [AI crawlers and crawlability](../ai-search/ai-crawlers.md) — the roster to admit, and robots.txt policy for each
- [The AI crawler registry](../appendix/crawler-registry.md) — UA strings, operators, verification methods
- [How AI finds and cites](../foundations/ai-retrieval.md) — why answer-time fetchability gates citations
- [Reverse proxies and CMS traps](reverse-proxy-cms.md) — the other family of "crawlers see something else" bugs
- [The 30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) — the detection battery as a routine
- Source skill: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema) (the WAF-invisibility diagnosis rule)
