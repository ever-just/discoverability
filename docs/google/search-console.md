# Google Search Console

**Search Console (GSC) is the only ground truth for how Google sees your site — set it up and record a baseline before you change anything.** Everything else in this book that touches Google assumes you have it: it's where you learn whether pages are indexed or silently discarded, which queries you actually appear for, and whether a fix worked. This chapter covers property types, every verification method including the fully API-driven service-account route we've shipped twice, the URL Inspection API as a diagnostic, the Indexing API's real (narrow) scope, and how to read the reports without fooling yourself.

## Property types: URL-prefix vs Domain

| | URL-prefix property | Domain property (`sc-domain:`) |
|---|---|---|
| Covers | Exactly one protocol + host + path prefix (`https://www.example.com/`) | **All** subdomains and protocols of the domain (`example.com`, `www.`, `app.`, `docs.`, http + https) |
| Verification | HTML file, meta tag, GA/GTM snippet, or DNS | **DNS TXT record only** |
| Use when | You can't touch DNS, or you want to scope reporting to one section | Almost always otherwise |

**Use a Domain property when you control DNS.** One `sc-domain:example.com` property covered a marketing site on the apex, a docs/app subdomain, and an admin subdomain in a single view — no per-host re-verification, no "which property has that page" confusion. Both production setups described below are Domain properties.

## Verification methods

Verification proves ownership to Google. Methods, from simplest to most automatable:

1. **DNS TXT record** (Domain or URL-prefix) — GSC gives you a `google-site-verification=...` value; add it as a TXT record at the domain apex. Works for `sc-domain:` properties, survives site rebuilds, and is the method the API route below automates.
2. **HTML meta tag** (URL-prefix only) — `<meta name="google-site-verification" content="...">` in the `<head>`. Many CMSs have a field for it (on the everjust platform it's a single `google_search_console` field on the website record — the platform renders the meta tag for you).
3. **HTML file upload** (URL-prefix only) — a file at the site root. Fragile on platforms where deploys or CMS routing can drop root files; if you must use it, serve it from the web server/proxy layer, not an application route.
4. **Google Analytics / Tag Manager snippet** — works, but ties verification to your analytics install; avoid.

!!! warning "Verification is per-Google-account, not per-domain"
    Each Google account that wants verified access to a Domain property needs **its own** TXT token. We verified a domain via a service account's token, added the human owner as a delegated owner via the API — and their own GSC UI *still* demanded verification until their account's own TXT record was added alongside the first. Multiple `google-site-verification` TXT records coexist safely. **Read the existing TXT record set before adding** — TXT upserts that replace the whole record set have clobbered SPF records in the wild (shipped-and-verified lesson; one deployment's health harness has a check that exists solely to prove the SPF record survived a verification write).

## The fully API-driven route (service account)

Shipped and verified twice (July 2026) on production domains. Use this when an agent or script needs to stand up GSC with no human clicking through a console. Pattern:

**Prerequisites:** a GCP project with the **Site Verification API** and **Search Console API** enabled; a service account (SA); programmatic access to the domain's DNS zone. Keep the SA key out of the repo, restrict file permissions, and prefer a dedicated project. If org policy blocks SA keys entirely, the keyless variant is **impersonation**: a human/business account holding the token-creator role on the SA mints short-lived tokens for it (allow a minute or two for IAM propagation).

1. **Mint a scoped token.** OAuth2 JWT-bearer flow with scopes `https://www.googleapis.com/auth/siteverification` + `https://www.googleapis.com/auth/webmasters`. No SDK required — you can sign the RS256 JWT with openssl and POST it to Google's token endpoint.
2. **Get a verification token:** `POST siteVerification/v1/token` with `{"site": {"type": "INET_DOMAIN", "identifier": "example.com"}, "verificationMethod": "DNS_TXT"}` → returns the TXT value.
3. **Place the TXT record** via your DNS provider's API (merge with existing TXT records — see the warning above).
4. **Self-verify:** `POST siteVerification/v1/webResource` with the same site object. The SA is now a verified owner of the domain.
5. **Delegate to humans:** `PUT siteVerification/v1/webResource/{id}` with an `owners` list including the business account (and, for client work, the client's own account — the property belongs to *their* business). Gotcha: use the webResource `id` **exactly as the API returned it** — it's already URL-encoded, and encoding it again yields an "invalid site" error.
6. **Add the GSC property:** `PUT webmasters/v3/sites/sc-domain%3Aexample.com` → HTTP 204.
7. **Submit the sitemap:** `PUT webmasters/v3/sites/{site}/sitemaps/{encoded-sitemap-url}` → 204. Confirm with the sitemap list call; in one run Google had already started fetching the sitemap the same night.

**Scope and auth gotchas (all hit in production):**

- `webmasters.readonly` can run Performance queries but **sitemap submission returns 403** — writes need the full `webmasters` scope.
- **API keys never work on these APIs** — you get 401 "API keys are not supported by this API". A Maps key is not a GSC credential. OAuth (SA or user) only.
- Account hygiene: verify business properties with the **business** account/SA, not whatever personal Gmail happens to be signed in. We had to remove a personal account added as owner by convenience — treat owner lists as an access-control surface.

**How you know it worked:** the property appears in `GET webmasters/v3/sites` with `siteOwner` permission; the sitemap shows in the sitemaps list with a last-downloaded timestamp; and a Performance query returns rows (within days on an established site; new domains take longer to show data).

## URL Inspection API: the ground-truth diagnostic

`POST https://searchconsole.googleapis.com/v1/urlInspection/index:inspect` with `{"inspectionUrl": ..., "siteUrl": "sc-domain:example.com"}` tells you what Google *actually* holds for a URL — not what you hope it holds:

- **`coverageState` / `indexingState`** — the verdict. The single most valuable diagnostic we've run: every page of a live business site returned `coverageState: "Excluded by 'noindex' tag"`, `indexingState: BLOCKED_BY_META_TAG` — while `lastCrawlTime` showed Googlebot visiting **daily** and discarding everything. Root cause was a CMS/reverse-proxy interaction emitting `noindex` sitewide ([the trap catalog](sitemaps-and-robots.md#the-noindex-trap-catalog)). Weeks of content work were moot until this one field was read.
- **`lastCrawlTime`** — proves crawling is happening (or not). Crawled-daily-but-excluded and never-crawled are entirely different problems.
- **Canonical fields** — Google-selected vs user-declared canonical; catches canonicals pointing at an internal host.
- **`richResultsResult`** — per-URL rich-result verdicts. This is how we found job pages indexed but failing JobPosting validation with `Missing field "description"` (ERROR) plus warnings — fixed the template, re-validated, then used the Indexing API (legitimately — see below) to push the recrawl.

Inspect a representative sample after any risky change: homepage, one page per template type, one recently published page. The API is quota-limited (documented at ~2,000 requests/day per property as of 2026) — sample, don't sweep.

## The Indexing API's REAL scope

The Google Indexing API is officially for **JobPosting and BroadcastEvent pages only**. That is the documented policy, and it has teeth:

- **Legitimate use** (shipped-and-verified): after repairing broken JobPosting markup on nine `/jobs/*` URLs, we pushed them through the Indexing API (`indexing` scope) for fast recrawl. That's the designed purpose.
- **Off-label use is a trap.** Submitting normal marketing pages "for instant indexing" — directly or via the many WordPress/SaaS plugins that wrap it — risks revocation and delivers no lasting benefit. Google has said as much, and there is no evidence it improves ranking or durable indexing for out-of-scope content.
- There is **no legitimate auto-submit channel to Google** for a normal site. Sitemap ping: removed 2023, endpoint 404s. IndexNow: not honored by Google. The GSC "Request Indexing" button: ~10–12/day quota, browser-session only, can't be automated headlessly. Google's freshness loop for you is: verified property + submitted sitemap + robots `Sitemap:` line → Google re-crawls forever.

## Reading the reports

- **Performance** — clicks, impressions, CTR, average position, by query/page/country/device. The *queries* table is the strategic gold: it tells you which intents Google already associates with you and at what position. Query rows at positions 8–20 are "striking distance" — the raw material for [keyword strategy](keyword-strategy.md). Query rows at position 40+ are Google saying "relevant, but you haven't earned it yet."
- **Pages (indexing)** — indexed vs excluded, with reasons. "Crawled — currently not indexed" on a young site is normal patience; "Excluded by noindex" or "Duplicate without user-selected canonical" is a defect to fix today.
- **Sitemaps** — submitted vs discovered URL counts and fetch status. Watch the URL count as a ledger: one product site's sitemap went 93 → 101 → 117 across three publishing waves, resubmitted each time; a mismatch between what you shipped and what the sitemap advertises means [a stale sitemap cache](sitemaps-and-robots.md).
- **Enhancements / rich results** — per-type validity counts once you ship structured data ([monitoring](rich-results.md#validation-and-monitoring)).

Expectation-setting for new domains (documented + observed): first pages indexed in ~2–4 weeks, fuller coverage ~2–3 months. Impressions precede clicks by weeks. Don't panic-tweak inside that window.

## Baseline discipline: record before changing

**Before any SEO change, snapshot 90 days of Performance data.** It takes one API call and makes every later claim honest. Real baselines from the case studies:

| Property (2026-07) | Clicks / Impressions (90d) | Avg position | Reading |
|---|---|---|---|
| customdomain.ai (07-15) | 5 / 159 | 42.5 | Indexed, not penalized, effectively invisible; impressions on the *right* intents at unrankable positions. Verdict: young/low-authority — a content+authority problem, not a technical one. |
| headsupoutdoorservices.com (07-29, pre-fix) | 24 / 640 | 14.5 | Almost entirely the brand term at position ~1.8; commercial terms nowhere. Diagnosed the same day as sitewide noindex — the baseline is what proves the fix worked later. |

Without the "before", you cannot distinguish "our fix worked" from "Google drifted." With it, re-measure at 2–4 weeks and let the delta speak. This discipline is cheap and non-optional; it's expanded in [Measurement](../foundations/measurement.md).

## Gotchas

- **Each Google account needs its own verification TXT** — API verification + owner delegation does not light up the owner's own UI. Add their token too; tokens coexist.
- **Read-modify-write DNS TXT records.** A whole-record-set upsert can silently destroy SPF or other tokens sharing the name.
- **`webmasters.readonly` 403s on sitemap submit** — request the write scope up front.
- **API keys are not credentials here** — 401 regardless of key validity. OAuth only.
- **Use the webResource id as returned** — double-URL-encoding it fails with "invalid site".
- **Impersonation propagation** — keyless SA impersonation needs the token-creator grant and ~1–2 minutes of IAM propagation; also, your CLI's "active account" display can differ from the account actually configured — trust the config value.
- **Don't grant personal accounts on business properties** — and for client properties, do add the *client's* account as owner: it's their asset.
- **Crawled ≠ indexed.** `lastCrawlTime` moving while pages stay excluded means Google is fetching and discarding — go read the exclusion reason, not more content advice.
- **A property with no data isn't broken** — new properties backfill within days; Domain properties only show data collected *after* Google associates crawls with the verified property.

## Related

- [Sitemaps and robots.txt](sitemaps-and-robots.md) — what to submit, and the traps that make GSC report horrors
- [Rich results](rich-results.md) — the enhancement reports and per-URL rich-result diagnostics
- [Keyword and SERP strategy](keyword-strategy.md) — turning Performance queries into a plan
- [Measurement and baselines](../foundations/measurement.md) — the wider measurement stack
- [Bing Webmaster Tools](../bing/bing-webmaster-tools.md) — the equivalent console for the index that feeds ChatGPT
- [Technical: reverse proxies and CMS traps](../technical/reverse-proxy-cms.md) — root causes behind the worst GSC readings

Source skills: [everjust-website-seo](https://github.com/ever-just/agentskills/tree/main/skills/everjust-website-seo) · [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization)
