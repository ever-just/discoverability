# Technical: the infrastructure layer

**Infrastructure decides your visibility before your content gets a vote.** The two worst discoverability failures in this book's source material were not content problems: a WAF bot-challenge that returned 429 to every crawler (`site:` showed **zero indexed pages** while the site looked perfect to humans), and a single CMS domain field that silently stamped `noindex` on every page of a live site for 11 days. Both were invisible from inside the CMS, the analytics, and the editor. Both were found the same way: **by testing the public surface the way crawlers see it.**

That is the meta-lesson of this whole part. Your hosting platform, DNS setup, reverse proxy, WAF, and rendering strategy each hold a veto over everything the other parts of this book teach. Schema, content clusters, and GEO writing are worth nothing to a crawler that gets a challenge page, a `Disallow: /`, or a canonical pointing at an internal hostname.

## The five failure families

| Family | What silently breaks | Chapter |
|---|---|---|
| **Domains & DNS** | You try to write DNS records at the registrar, but the zone actually lives somewhere else. Verification TXTs, email records, and cutovers all stall on "no zone" errors. | [Domains and DNS](domains-and-dns.md) |
| **Email trust** | Your domain sends mail the world distrusts — no SPF/DKIM/DMARC, or misaligned ones. Outreach, review requests, and reports go to spam; the domain's trust posture looks neglected. | [Email trust](email-trust.md) |
| **Reverse proxies & CMS traps** | The CMS sees an internal hostname, not your public domain, and builds robots.txt, sitemaps, canonicals — and indexability decisions — from the wrong host. | [Reverse proxies and CMS traps](reverse-proxy-cms.md) |
| **Domain migrations** | You cut over to a new domain but the old one survives in link bases, canonicals, sitemaps, signatures, and emails — or the cutover itself ships a fresh indexing bug. | [Domain migrations](domain-migration.md) |
| **Rendering & WAFs** | Your security posture (bot challenges, rate limits) or client-side rendering serves crawlers something other than your content. Humans see nothing wrong. | [Rendering, WAFs, and bot challenges](rendering-and-waf.md) |

## Symptom → chapter

Start from what you're seeing:

| Symptom | Go to |
|---|---|
| Registrar's DNS API returns "no zone" / "domain not found" for a domain you own | [Domains and DNS](domains-and-dns.md) |
| `site:yourdomain.com` returns zero results; social profiles outrank your own domain | [Rendering & WAFs](rendering-and-waf.md) |
| Search Console says pages are `Excluded by 'noindex' tag` you never wrote | [Reverse proxies and CMS traps](reverse-proxy-cms.md) |
| robots.txt says `Disallow: /` and you didn't put it there | [Reverse proxies and CMS traps](reverse-proxy-cms.md) |
| Sitemap or canonical URLs show a hostname that isn't your public domain | [Reverse proxies and CMS traps](reverse-proxy-cms.md) |
| Your emails land in spam, or a mail provider rejects your domain | [Email trust](email-trust.md) |
| You're rebranding / moving domains and want to keep your indexing | [Domain migrations](domain-migration.md) |
| An SEO edit "shipped" but the live page doesn't show it | [Reverse proxies and CMS traps](reverse-proxy-cms.md) |
| ChatGPT/Perplexity can't fetch your pages even though Google can | [Rendering & WAFs](rendering-and-waf.md) |

## The one habit that catches all of it

Every failure in this part was caught (or would have been caught sooner) by the same 2-minute probe, run **against the public domain, not a staging or internal host**:

```bash
# Fetch as a crawler, not as yourself
curl -sS -o /dev/null -w "%{http_code}\n" -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://yourdomain.com/
curl -sS -A "GPTBot/1.1" https://yourdomain.com/robots.txt
curl -sS -A "GPTBot/1.1" https://yourdomain.com/sitemap.xml | head -20

# Then read what the HTML actually says
curl -sS https://yourdomain.com/ | grep -iE 'name="robots"|rel="canonical"'
```

Four checks: status codes as a bot, robots.txt content, sitemap hostnames, and the meta-robots + canonical tags in live HTML. If you only adopt one practice from this part, make it running this probe after **every** infrastructure change — DNS flips, proxy edits, CMS settings, security toggles — and on a monthly cadence. The [30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) builds this into a full routine.

!!! warning "Dashboards lie by omission"
    Your CMS admin, your uptime monitor, and your own browser all see the site through paths crawlers don't take. A site can serve 200 with perfect content to every human while serving 429 challenge pages, `noindex` tags, or internal hostnames to every bot. None of the incidents in this part triggered a single alert.

## How this part is organized

Read in order if you're setting up new infrastructure — DNS first, then email, then the serving path. Jump straight to the matching chapter if you're diagnosing. Every chapter ends with verification commands, a gotchas section drawn from real production incidents, and links to the reusable skills the material came from.

## Related

- [How discovery works in 2026](../start/how-discovery-works.md) — where crawlability sits in the three-surface model
- [Sitemaps and robots.txt](../google/sitemaps-and-robots.md) — the directive files these infrastructure traps corrupt
- [AI crawlers and crawlability](../ai-search/ai-crawlers.md) — the bot roster your infrastructure must admit
- [The 30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) — the probe above, expanded into a routine
- [everjust.app tenants case study](../case-studies/everjust-tenants.md) — where most of these scars come from
