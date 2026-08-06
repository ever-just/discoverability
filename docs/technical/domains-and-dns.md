# Domains and DNS

**The registrar you bought a domain from and the DNS host that answers queries for it are two different roles, often held by two different companies — and only the DNS host can write records.** Before you attempt any DNS change, run `dig NS yourdomain.com` and believe what it says over what your billing statement says. Half the "DNS won't configure" failures we've diagnosed reduce to writing records at the wrong company.

This chapter covers the registrar-vs-DNS-host split, the record types that matter for discoverability, and how to automate registrar APIs without destroying a zone.

## Registrar ≠ DNS host

A domain involves two independent jobs:

| Role | What it does | How to identify it |
|---|---|---|
| **Registrar** | Holds the registration: renewal, ownership, WHOIS contacts, and — crucially — the pointer to the nameservers | `whois yourdomain.com \| grep -i 'registrar:'` |
| **DNS host** | Runs the nameservers that actually answer queries; the only place records can be written | `dig +short NS yourdomain.com` |

They're the same company by default (buy at GoDaddy, use GoDaddy DNS) but diverge the moment anyone delegates nameservers — which every "connect your domain to Vercel/Netlify/Cloudflare/Shopify" flow does. After delegation, the registrar's DNS panel and DNS API are **dead weight for that domain**: the zone doesn't live there anymore.

**Real case (shipped and verified, 2026-07):** a domain registered at GoDaddy had its nameservers delegated to Vercel years earlier. The GoDaddy DNS API returned "no zone" for it. That error is the *diagnosis*, not a bug to retry — the records had to be written at Vercel. The registrar was only useful for one thing: changing the nameserver delegation itself.

### The 60-second diagnosis walkthrough

```bash
# 1. Who is authoritative? (the DNS host — where records get written)
dig +short NS yourdomain.com
# e.g. ns1.vercel-dns.com.  → records live at Vercel
# e.g. ns09.domaincontrol.com.  → GoDaddy IS the DNS host

# 2. Who is the registrar? (renewal + delegation only)
whois yourdomain.com | grep -iE 'registrar:|name server'

# 3. Sanity-check that the authoritative server actually answers
dig @$(dig +short NS yourdomain.com | head -1) yourdomain.com A +short
```

Decision rule:

- `dig NS` and the registrar **match** → manage DNS through the registrar (its API works; see automation below).
- They **differ** → write every record at the company `dig NS` names. Use the registrar only to renew or to re-delegate nameservers. Expect the registrar's DNS API to error with "no zone" / "domain not found" — that's correct behavior.
- **Never change nameservers when you only meant to add a record.** Re-delegation moves the entire zone (site, email, verifications) and is the single most destructive "small change" available in a DNS panel.

## Record types that matter for discovery

Everything a crawler, answer engine, or agent learns about your domain before fetching a page comes through DNS:

| Record | Discoverability role |
|---|---|
| `NS` | Who controls everything below. Always check first. |
| `A` / `AAAA` | Where the site lives. The record you flip in a [domain migration](domain-migration.md). |
| `CNAME` | Subdomain aliases (`www`, `docs`, `app`) to platform hosts. Most registrars refuse CNAME at the apex — use an A record or a DNS host with CNAME flattening (e.g. Cloudflare). |
| `TXT` (verification) | `google-site-verification=...` for [Search Console](../google/search-console.md), Bing verification, domain-validation for certificates (ACME DNS-01), and DNS-based namespace proof for the [MCP Registry](../agents/mcp-registry.md). |
| `TXT` (email auth) | SPF, DKIM, DMARC — the whole of [Email trust](email-trust.md). |
| `MX` | Where your mail is delivered. Independent of the site records (see below). |
| `CAA` | Which certificate authorities may issue for the domain. A forgotten CAA record can block cert issuance mid-migration. |
| `SVCB` / `HTTPS`, agent TXT records | The emerging agent-discovery layer — see [Manifests and DNS-AID](../agents/manifests-and-dns.md). |

Three operational facts about these records, all verified in production:

1. **Multiple TXT records on the same name coexist.** A domain can hold several `google-site-verification` tokens (verification is per-Google-account — a service-account verification does not satisfy the owner's personal account; each needs its own token) alongside its SPF record. **Read the existing TXT set before any upsert** so you never clobber SPF while adding a verification token.
2. **Email records live in a different namespace than site records.** Adding MX and TXT cannot break the A/CNAME records serving your site — you can add email DNS to a live domain with zero risk, *as long as you add rather than replace* (see the PUT trap below). Full treatment in [Email trust](email-trust.md).
3. **TTL is your rollback insurance.** Before any planned change, lower the affected records' TTL (300–600s) at least one old-TTL-period in advance, so a bad flip can be reverted in minutes instead of hours.

## Registrar-API automation: the pattern

Registrar and DNS-host APIs (GoDaddy, Cloudflare, Route 53, Name.com, …) share one shape and one lethal trap. Using GoDaddy's API as the worked example (documented behavior from its official spec, as of 2026-08):

**The trap: add-semantics vs replace-semantics.**

| Verb pattern | Effect | Risk |
|---|---|---|
| `PATCH /records` | **Appends** records to the zone | Safe — cannot delete anything |
| `PUT /records/{type}/{name}` | Replaces only the records matching that type+name | Contained — the right tool for updating one record |
| `PUT /records` (whole zone) | **Replaces the entire zone** — anything not in the request body is deleted | Can silently destroy the site's A records, the mail MX, every verification TXT |

The doctrine that follows: **read–merge–upsert, never blind-replace.** Fetch the current records for the exact type+name you're changing, merge your change, and write back at the narrowest scope the API offers. We adopted this rule after finding a zone whose apex carried the owner's live personal-inbox MX record — a whole-zone PUT prepared from "our" records alone would have severed their personal email.

Other registrar-API realities worth planning around (GoDaddy-specific values; every provider has equivalents):

- **API access is entitlement-gated.** GoDaddy restricted DNS API access in 2024 (requires ≥1 domain on the account; availability checks require 50+). A working key on one account can 403 on a domain held in a *different* account — client domains often live in client accounts. Test with a harmless read (or a no-op PATCH) before building automation on an assumption.
- **TTL minimums** (600s at GoDaddy — a lower value 422s).
- **`@` denotes the zone apex** in record names.
- **NS and SOA can't be deleted** via the records API.
- **2FA blocks some operations via API** — nameserver changes on "protected" domains may require the web UI. Your cutover runbook needs a manual fallback step.
- **Rate limits** (~60 req/min/endpoint at GoDaddy): batch multiple records into one call rather than looping.

One more registrar asymmetry that decides migration architecture: **some registrars cannot pre-stage a zone while nameservers are delegated elsewhere** (GoDaddy is one, verified 2026-07). If you plan to move DNS *to* such a registrar as part of a cutover, the zone goes live empty-or-parked the moment NS switches — a live site would flash a parked page. The fix is to pre-stage the full zone at a host that allows it (Route 53, Cloudflare) and delegate to *that* instead. Details in [Domain migrations](domain-migration.md).

## Verification: prove every change from the outside

Never trust a DNS panel's "saved" state — query the public resolvers:

```bash
dig +short A yourdomain.com          # site target
dig +short CNAME www.yourdomain.com
dig +short TXT yourdomain.com        # verifications + SPF, all together
dig +short MX yourdomain.com
dig +short TXT _dmarc.yourdomain.com

# Ask the authoritative server directly to bypass propagation lag
dig @ns1.your-dns-host.com yourdomain.com A +short
```

Propagation is TTL-bound: the authoritative answer changes instantly; resolvers serve the old answer until their cached TTL expires. When a change "hasn't propagated," compare the authoritative answer to `dig` without `@` — if the authoritative one is right, you're done and the rest is waiting.

## Gotchas

1. **Assuming the registrar controls DNS.** The #1 failure. `dig NS` first, always. A registrar-API "no zone" error means delegation, not breakage.
2. **Whole-zone PUT as a "clean write."** It deletes every record you didn't include — site A records, someone's live MX, all verifications. Read–merge–upsert at the narrowest scope.
3. **Clobbering SPF with a verification TXT.** Panels and APIs that "set" TXT at the apex instead of "adding" have wiped SPF records. List existing TXTs first.
4. **Changing nameservers to add one record.** Re-delegation moves everything. Add the record at the current DNS host instead.
5. **Assuming one Google verification covers all accounts.** Verification is per-Google-account; the owner's own account needs its own TXT token even after a service account verified the domain (hit and fixed 2026-07).
6. **High TTLs discovered mid-incident.** A 24h TTL on the record you need to roll back means a 24h outage window. Lower TTLs *before* planned changes.
7. **CAA records blocking cert issuance.** If Let's Encrypt (or your CA) suddenly can't issue during a migration, check `dig CAA yourdomain.com` before debugging the ACME client.
8. **Building automation against an untested account/key pairing.** Keys are account-scoped; client domains live in client accounts; entitlements gate endpoints. One harmless read verifies the whole chain.

## Related

- [Email trust (SPF/DKIM/DMARC)](email-trust.md) — the email half of your DNS zone
- [Domain migrations](domain-migration.md) — where DNS control, pre-staging, and TTL strategy decide success
- [Google Search Console](../google/search-console.md) — the DNS-TXT verification flow this chapter's records support
- [Manifests and DNS-AID](../agents/manifests-and-dns.md) — DNS as an agent-discovery surface
- Source skills: [custom-domain-email-dns-diagnosis](https://github.com/ever-just/agentskills/tree/main/skills/custom-domain-email-dns-diagnosis), [godaddy-api](https://github.com/ever-just/agentskills/tree/main/skills/godaddy-api)
