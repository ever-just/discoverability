# Tool directory

Every external tool, console, API, and data source the book references, grouped by job. One line each: what it's for, where it lives, what it costs. Free tiers noted as of 2026-08 — pricing and quotas drift, so re-check before building automation on any of them.

## Google

| Tool | What it's for | URL | Cost |
|---|---|---|---|
| Google Search Console | The first-party truth source for Google visibility: clicks, impressions, position, index coverage, sitemap submission. Record baselines here **before** any change | [search.google.com/search-console](https://search.google.com/search-console) | Free |
| Search Console API | Automate GSC: list properties, query performance data, submit sitemaps — drivable by a service account with `siteOwner` permission (works on `sc-domain:` properties) | [developers.google.com/webmaster-tools](https://developers.google.com/webmaster-tools) | Free (write calls need the full `webmasters` scope — `webmasters.readonly` can query but 403s on sitemap submits) |
| Site Verification API | Verify property ownership programmatically: request a DNS TXT token, publish it, confirm — then delegate ownership to human accounts with one API call | [developers.google.com/site-verification](https://developers.google.com/site-verification) | Free |
| URL Inspection API | Per-URL diagnosis as Google sees it: index state, `BLOCKED_BY_META_TAG`-class exclusions, detected canonical, rich-result errors | [developers.google.com/webmaster-tools/v1/urlInspection.index/inspect](https://developers.google.com/webmaster-tools/v1/urlInspection.index/inspect) | Free, quota-capped (~2,000 calls/day/property) |
| Indexing API | Fast recrawl requests — **for `JobPosting` and `BroadcastEvent` pages only**. Off-label use on normal pages risks revocation; the old sitemap-ping endpoint is dead (removed 2023) | [developers.google.com/search/apis/indexing-api](https://developers.google.com/search/apis/indexing-api/v3/quickstart) | Free |
| Rich Results Test | Validates whether a live URL or snippet is eligible for Google's *visual* rich results. Shows only Google-rendered types — Service/OfferCatalog/DefinedTerm parsing silently passes elsewhere | [search.google.com/test/rich-results](https://search.google.com/test/rich-results) | Free |
| Google Business Profile | The local-business listing behind the map pack and AI local recommendations. Manage in the browser; the Business Profile API is approval-gated (default quota 0 — apply weeks early) | [business.google.com](https://business.google.com) | Free (API approval required) |
| PageSpeed Insights | Field + lab performance data (Core Web Vitals, LCP, compression) for any public URL; has a free API | [pagespeed.web.dev](https://pagespeed.web.dev) | Free |
| Google Postmaster Tools | Domain-level Gmail deliverability: spam rate, domain/IP reputation, DMARC failures. The monitoring half of the email-trust work | [postmaster.google.com](https://postmaster.google.com) | Free |

## Bing

| Tool | What it's for | URL | Cost |
|---|---|---|---|
| Bing Webmaster Tools | Bing's console — disproportionately important because the Bing index feeds ChatGPT search and Copilot. Can import verified properties straight from GSC. Its AI Performance report is the only place to observe Copilot/ChatGPT-side reception | [bing.com/webmasters](https://www.bing.com/webmasters) | Free |
| IndexNow | Push protocol: POST changed URLs (with a self-hosted key file) and Bing, Yandex, Seznam, and Naver pick them up. **Google ignores it.** Accelerates crawl, does not rank or cite | [indexnow.org](https://www.indexnow.org) — endpoint `https://api.indexnow.org/indexnow` | Free |

## Schema validation

| Tool | What it's for | URL | Cost |
|---|---|---|---|
| Schema Markup Validator | The full schema.org parser: validates *any* JSON-LD graph, catches dangling `@id` refs and wrong-node properties that Rich Results Test never shows | [validator.schema.org](https://validator.schema.org) | Free |
| Rich Results Test | The Google-eligibility half of validation (see Google section). Run both: RRT for "will Google render it", the validator for "does the graph parse" | [search.google.com/test/rich-results](https://search.google.com/test/rich-results) | Free |

Third check, always: `curl` the **live page** and grep for `application/ld+json` — DB-verified is not rendered-verified.

## DNS and email

| Tool | What it's for | URL | Cost |
|---|---|---|---|
| `dig` | The DNS ground-truth tool. `dig NS example.com` before assuming who can write records; `dig TXT`, `MX`, `+dnssec` for the rest | Ships with BIND utilities (preinstalled on macOS/Linux) | Free |
| Cloudflare DoH (JSON) | `dig` substitute that works anywhere HTTPS does: `curl -H 'accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=example.com&type=TXT'`. Status 3 = NXDOMAIN; the AD flag shows DNSSEC | [cloudflare-dns.com/dns-query](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-https/) | Free |
| mail-tester | Send a test email, get a 0–10 deliverability score with SPF/DKIM/DMARC breakdown. Send a *realistic* body — short image-heavy test bodies get docked on content ratio | [mail-tester.com](https://www.mail-tester.com) | Free (few checks/day), paid for more |
| parsedmarc | Open-source DMARC aggregate/forensic report parser — turns the raw XML your `rua=` mailbox receives into readable reports and dashboards | [github.com/domainaware/parsedmarc](https://github.com/domainaware/parsedmarc) | Free (OSS) |
| Hosted DMARC analyzers | Managed rua/ruf ingestion and alignment dashboards (dmarcian, EasyDMARC, Postmark's free tool) if you'd rather not self-host parsing | e.g. [dmarcian.com](https://dmarcian.com) | Freemium |
| Cloudflare API | DNS record CRUD, zone settings, and DNSSEC on Cloudflare-hosted zones — the write side of most record recipes in this book | [developers.cloudflare.com/api](https://developers.cloudflare.com/api/) | Free tier covers DNS |
| Amazon Route 53 | Programmable DNS with instant propagation — the pre-stageable zone used in the book's migration playbooks | [aws.amazon.com/route53](https://aws.amazon.com/route53/) | Paid (~$0.50/zone/mo + queries) |
| GoDaddy API | Registrar-side automation (NS changes, records). Record-write endpoints are entitlement-gated on some account tiers — verify access before you script against it | [developer.godaddy.com](https://developer.godaddy.com) | Free with account |

## Crawl and fingerprint

| Tool | What it's for | URL | Cost |
|---|---|---|---|
| `curl` as a bot | The one-line crawlability probe: fetch your site with each crawler's User-Agent and compare status codes (full pattern in the [crawler registry](crawler-registry.md)). Finds bot walls that make a site invisible while browsers see it fine | [curl.se](https://curl.se) | Free |
| httpx (ProjectDiscovery) | Bulk HTTP probing: status, titles, tech detection, TLS data across many hosts at once — the workhorse for fingerprinting a competitor's stack | [github.com/projectdiscovery/httpx](https://github.com/projectdiscovery/httpx) | Free (OSS) |
| Wappalyzer | Browser-extension / API tech-stack detection: CMS, CDN, analytics, ad pixels, ecommerce platform per page | [wappalyzer.com](https://www.wappalyzer.com) | Freemium |
| BuiltWith | Same job, longer memory — historical technology profiles show what a site *used* to run | [builtwith.com](https://builtwith.com) | Freemium |
| crt.sh | Certificate-transparency search: enumerate a domain's subdomains from issued TLS certs — infrastructure recon with zero touch of the target | [crt.sh](https://crt.sh) | Free |

## Traffic estimation

The zero-budget triangulation stack from [Measurement and baselines](../foundations/measurement.md) — no single source is trustworthy; agreement across them is.

| Tool | What it's for | URL | Cost |
|---|---|---|---|
| SimilarWeb free endpoint | `https://data.similarweb.com/api/v1/data?domain=example.com` — no auth, returns ~3 months of estimated visits, top keywords, and CPC. Unofficial and undocumented (as of 2026-08), so treat as fragile; also pull known competitors through it to calibrate | [data.similarweb.com](https://data.similarweb.com/api/v1/data?domain=example.com) | Free (paid product behind it) |
| Wayback CDX API | The Internet Archive's crawl index: capture frequency is a popularity proxy, and the URL inventory maps a site's evolution over years | [web.archive.org/cdx/search/cdx](https://archive.org/developers/wayback-cdx-server.html) | Free |
| Tranco | Research-grade domain-popularity ranking (a hardened blend of the old Alexa-style lists). Presence/absence and rank band, not visit counts | [tranco-list.eu](https://tranco-list.eu) | Free |
| Cloudflare Radar | Domain rank buckets, traffic and bot-share trends from Cloudflare's vantage point; its verified-bots directory doubles as a crawler-verification reference | [radar.cloudflare.com](https://radar.cloudflare.com) | Free |

## AI and agent surfaces

| Tool | What it's for | URL | Cost |
|---|---|---|---|
| MCP Registry | The canonical upstream listing for MCP servers — publish `server.json` once (via the `mcp-publisher` CLI + DNS namespace verification) and the community directories mirror it | [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io) — schema/CLI at [github.com/modelcontextprotocol/registry](https://github.com/modelcontextprotocol/registry) | Free |
| mcp.so | Community MCP directory agents and users browse; claim your mirrored listing | [mcp.so](https://mcp.so) | Free |
| Smithery | MCP directory + tooling (`smithery` CLI can publish hosted servers) | [smithery.ai](https://smithery.ai) | Free listing |
| Glama | MCP directory with server health/inspection views | [glama.ai/mcp/servers](https://glama.ai/mcp/servers) | Free listing |
| PulseMCP | MCP directory with weekly ecosystem coverage — claim the listing, don't just appear | [pulsemcp.com](https://www.pulsemcp.com) | Free listing |
| dns-aid | Reference implementation for DNS-AID: publishes and verifies the SVCB/TXT/`_index._agents`/TLSA record set (`pip install "dns-aid[cloudflare]"`, then `dns-aid publish` / `dns-aid verify`) | [github.com/infobloxopen/dns-aid-core](https://github.com/infobloxopen/dns-aid-core) | Free (OSS) |
| Anthropic Connectors Directory | First-party, human-reviewed listing for Claude connectors — requires per-tool annotations, a privacy policy URL, and a Team/Enterprise org to submit | [claude.ai](https://claude.ai) (admin settings) | Free to submit; org plan required |
| ChatGPT Apps submission | OpenAI's first-party, reviewed channel for ChatGPT-surfaced apps/connectors | [platform.openai.com](https://platform.openai.com) | Free to submit |

## Related

- [Measurement and baselines](../foundations/measurement.md) — the method behind the traffic-estimation stack
- [Google Search Console](../google/search-console.md) — the GSC + APIs workflow in depth
- [IndexNow](../bing/indexnow.md) — wiring the push protocol into a CMS
- [The MCP Registry](../agents/mcp-registry.md) and [Manifests and DNS-AID](../agents/manifests-and-dns.md) — the agent-surface tools in context
- [Email trust](../technical/email-trust.md) — where mail-tester, parsedmarc, and Postmaster Tools fit
- [AI crawler registry](crawler-registry.md) — the curl-as-bot pattern these probes rely on
