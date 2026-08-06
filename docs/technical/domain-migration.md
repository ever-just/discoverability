# Domain migrations

**Cutting a live site to a new domain is an inventory problem, then a sequencing problem.** The old domain doesn't live in one place — it's embedded in link bases, canonicals, sitemaps, page content, email identities, signatures, and proxy config, and any surface you miss keeps leaking the old identity (or worse, keeps serving the old site). The playbook: inventory every surface, pre-stage everything that can be pre-staged, flip DNS last with a rollback ready, then 301 the old domain — and **never** redirect the internal machine host that does your routing.

Two production cutovers ground this chapter: tcstartupweek.com (Vercel → self-hosted CMS, 2026-07-02) and headsupoutdoorservices.com (Vercel → self-hosted CMS, 2026-07-18). Both hit ~zero downtime. One still shipped a post-cutover indexing bug that lived for 11 days — the checklist below includes the step that would have caught it.

## Why migrate at all

The headsup case is the honest motivation: the owned .com sat behind a bot-challenge wall (invisible to search and AI — see [Rendering & WAFs](rendering-and-waf.md)) while the crawlable rebuilt site lived on a low-authority platform subdomain. Neither property could win: the good site couldn't inherit the brand's equity, and the brand domain blocked crawlers. Pointing the owned domain at the crawlable site was flagged as "the #1 SEO/AEO move — a decision, not code" three weeks before the owner green-lit it. If your situation rhymes, the migration *is* the SEO strategy.

## Step 0 — inventory every surface that embeds the old domain

Map before you touch. On a full-stack CMS, one domain lives in roughly nine places:

| Surface | If you miss it |
|---|---|
| **Link base** (the app's base-URL setting) | Every link inside every email the app sends points at the old host |
| **Canonical/SEO base** (the CMS domain field) | Sitemap, canonicals, og:url emit the old domain — but see the [reverse-proxy trap](reverse-proxy-cms.md) before setting this field |
| **Outbound mail identity** (sending domain, from-filter, catchall/bounce/default-from) | Mail sends from — or gets rejected on — the old domain |
| **User signatures** (stored HTML) | The old wordmark/link in every message a human sends — the #1 "it still says the old brand" report |
| **Page content + meta fields** (views, blog/event content, meta titles/descriptions) | Old brand text and links on the site itself |
| **Templates** (email templates) | Newly generated mail resurrects the old domain |
| **Calendar/video links** | Old host in standing meeting invites |
| **Proxy config** (vhosts for old + new) | The old domain literally keeps serving the site as a duplicate |
| **External registrations** (GSC/Bing properties, GBP website link, API-key referrer allowlists, ad/analytics config) | Tools measure and link the wrong property |

### The DB-wide sweep

Don't enumerate from memory — search the database for the old domain, **before** the cutover (to build the fix list) and **after** (to prove it's clean): config parameters, company records, user signatures and logins, mail templates, blog/event content, view markup, and every SEO meta field. Two hard-won rules:

- Translatable/JSON columns need casting to text before pattern-matching, or the query errors.
- **Leave historical sent mail alone.** Old messages legitimately contain the old domain — they're a record of what was sent. Rewriting them falsifies history. Fix only what *generates new* content. (The customdomain.ai rebrand swept to zero old-domain references everywhere except ~35 historical mail bodies, deliberately kept.)

## The cutover sequence

Pattern-level, in the order that worked twice. Each step ends with its verification.

**1. Recon — who controls what.** `dig NS` the domain ([registrar ≠ DNS host](domains-and-dns.md)); decide where the zone will live. Gotcha that shaped one migration: the registrar (GoDaddy) could not pre-stage a zone while nameservers pointed elsewhere — switching NS to it would have gone live on a parked page. DNS moved to a host that allows full pre-staging (Route 53), registrar demoted to registration-only.

**2. Pre-stage the complete DNS zone at the new DNS host.** Every record: the site records **pointing at the current (old) origin** so nothing changes at NS-switch time, plus **the entire email zone** — MX, SPF, DKIM, DMARC, bounce subdomain. Email records ride along unchanged; campaign sending survived one cutover untouched *because* the mail zone was copied first. Then switch nameservers: delegation goes live in minutes; resolver caches lag on old TTLs while the old host keeps serving — that's fine, it's serving the same site.

**3. Re-verify the mail identity.** ESP domain identities verify against DNS; one was found **silently FAILED for two months** (its verification had expired against a lapsed zone). Delete/recreate or re-verify the identity against the new zone *before* the flip, not after mail starts bouncing.

**4. Certificates before traffic.** Issue the cert for apex + `www` (+ any aliases) via **DNS-01 validation** — you control the zone now, so this works before any traffic moves, and the first visitor never sees a warning. Check `dig CAA` if issuance fails. (One box's system certbot was broken; an ACME client with DNS-API plugins did the job — have a fallback.)

**5. Proxy vhosts for the new domain.** New public domain → proxied to the app (with the Host rewrite your platform needs for tenant routing); `www` → apex 301; port 80 → 443 301; `admin.`/alias hosts marked noindex. Bake the [reverse-proxy robots/sitemap fixes](reverse-proxy-cms.md) into these blocks **now** — robots.txt and the sitemap regress *immediately* behind a Host-rewriting proxy, so the static-robots and sitemap `sub_filter` locations are part of the cutover, not a follow-up. Commit the config to the deploy source of truth so the next deploy doesn't revert it.

**6. Pre-flip gate.** Test the new path without moving DNS:

```bash
curl --resolve yourdomain.com:443:NEW_ORIGIN_IP -sSI https://yourdomain.com/ | head -5
curl --resolve yourdomain.com:443:NEW_ORIGIN_IP -sS https://yourdomain.com/robots.txt
```

Certificate, vhost routing, robots, and content — all verifiable pre-flip. One plan audit also caught that the proxy's catch-all would have served *a different tenant's site* during propagation; the pre-flip gate is where you find that class of bug.

**7. Flip DNS at a low TTL.** Drop the site records' TTL (300–600s) ahead of time; flip the A/CNAME to the new origin. **Rollback = flip those records back inside the new zone. Never roll back by reverting nameservers** — the zone now carries your mail and verifications; an NS revert orphans all of it.

**8. CMS cutover.** Set the app's base URL to the new domain **and freeze it** (unfrozen, some CMSs rewrite it to whatever host an admin last logged in from); handle the CMS domain field per the [reverse-proxy trap](reverse-proxy-cms.md); run the sweep fixes (signatures, templates, content, meta); **delete the cached sitemap** so it regenerates on the new domain.

**9. Retire the old domain — add path-preserving 301s.** The old domain keeps its vhost and its certificate (you must terminate TLS to redirect HTTPS), but every request returns `301 https://newdomain$request_uri` — apex and www, both ports. This is the add-new-*then*-retire-old order: the new domain was serving and verified before the old one started redirecting.

**10. The boundary: never 301 the internal machine host.** On platform architectures, the internal `tenant.platform.example` host is plumbing — tenant routing resolves on it, OAuth callbacks and mail webhooks land on it, links in already-sent mail point at it. It must keep answering 200 forever (noindexed, per the [duplicate-host fix](reverse-proxy-cms.md)). Once the brand surfaces move, users never see it — it does not need to "migrate."

**11. Verify the full surface.**

```bash
curl -so /dev/null -w '%{http_code} %{redirect_url}\n' https://olddomain.com/some/path   # 301 → new, path preserved
curl -so /dev/null -w '%{http_code}\n' https://newdomain.com/                            # 200
curl -sS https://newdomain.com/robots.txt                                                # your directives, public host
curl -sS https://newdomain.com/sitemap.xml | grep -c newdomain.com                       # locs on the NEW domain
curl -sS https://newdomain.com/ | grep -iE 'name="robots"|rel="canonical"'               # no noindex; canonical = new
curl -so /dev/null -w '%{http_code}\n' https://tenant.platform.example/                  # machine host still 200
```

Plus one **real transaction**: the headsup cutover's final proof was a live form submit on the new domain creating an actual CRM lead. Infrastructure that serves pages but drops conversions is not migrated.

**12. Post-cutover registrations.** Verify the new domain in [Search Console](../google/search-console.md) (DNS TXT; remember each Google account needs its own token) and submit the sitemap; use GSC's **change-of-address** when the old domain also had a property; update Bing, the GBP website link, and any API-key referrer allowlists that named the old hosts.

## The lesson that earns its own heading: keep watching after the flip

The headsup migration verified clean on cutover night — and still shipped sitewide `noindex` for 11 days, because a CMS domain field interacted with the proxy's Host rewrite ([the full incident](reverse-proxy-cms.md)). Cutover-night checks catch cutover-night bugs; **schedule a day-2 and week-2 re-probe** of the crawler-facing surface (the meta-robots grep above, plus GSC URL Inspection on key pages) so a delayed trap can't run for weeks. Record your GSC baseline before the migration so the recovery curve is measurable ([Measurement](../foundations/measurement.md)).

## Gotchas

1. **Rolling back by reverting NS.** Orphans the mail zone and verifications you just staged. Roll back A/CNAME records inside the new zone.
2. **Registrars that can't pre-stage a zone.** Check before planning; otherwise the NS switch parks your live domain. Pre-stage at Route 53/Cloudflare instead.
3. **301ing the machine host.** Breaks tenant routing, OAuth, webhooks, and every link in mail already sent. Noindex it; never redirect it.
4. **The expired mail identity.** Verification state rots while domains move. Check it pre-flip (step 3) — a 5xx-class send failure after cutover is usually an identity/policy problem at the ESP, not the CMS.
5. **Base URL unfrozen.** It silently flips to an admin's login host and every emailed link follows. Freeze it.
6. **Robots/sitemap regressions the moment the proxy fronts the new domain.** Part of the cutover (step 5), not cleanup.
7. **The stale cached sitemap** advertising the old domain until deleted (step 8).
8. **Sweep-by-memory.** The DB-wide search finds surfaces nobody remembered (calendar links, digest names, meta fields). Run it before and after.
9. **Trusting a green deploy or one curl.** Deploy badges lie about per-tenant outcomes, and a single 503 can fake a regression — verify the actual public surface, sequentially, with a re-fetch on failure.
10. **Forgetting the email-link base.** Post-cutover, tracked/unsubscribe links kept minting on the old base until it moved — a deliverability and brand-trust leak ([Email trust](email-trust.md)).

## Related

- [Reverse proxies and CMS traps](reverse-proxy-cms.md) — the trap family every proxied cutover must defuse
- [Domains and DNS](domains-and-dns.md) — NS control, pre-staging, TTL strategy
- [Email trust](email-trust.md) — the mail zone you carry through unchanged
- [Google Search Console](../google/search-console.md) — verification and change-of-address
- [everjust.app tenants case study](../case-studies/everjust-tenants.md) — both cutovers in narrative form
- Source skills: [reverse-proxy-cms-indexing](https://github.com/ever-just/agentskills/tree/main/skills/reverse-proxy-cms-indexing), [custom-domain-email-dns-diagnosis](https://github.com/ever-just/agentskills/tree/main/skills/custom-domain-email-dns-diagnosis), [everjust-tenant-domain-migration](https://github.com/ever-just/agentskills/tree/main/skills/everjust-tenant-domain-migration)
