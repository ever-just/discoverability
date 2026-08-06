# Email trust (SPF/DKIM/DMARC)

**SPF, DKIM, and DMARC decide whether the world's inboxes trust mail from your domain — and since 2024–2025 they are mandatory, not optional, for reaching Gmail and Outlook at all.** They also form a public, queryable trust posture for your domain: anyone (including vendors, partners, and automated systems) can read in two `dig` commands whether a domain's email is authenticated or spoofable. And one fact removes most of the fear from setting them up: **email records are independent of site records — adding MX and TXT cannot break a live website.**

Discoverability runs on email more than it looks like it does: outreach that earns the off-site mentions AI engines cite, review-request emails that build the ratings local AI recommendations key on, and the DMARC reports that tell you who's sending as you. A domain that lands in spam loses all of it.

## The independence principle

The records that serve your website (`A`, `AAAA`, `CNAME`) and the records that handle your email (`MX`, plus `TXT` for SPF/DKIM/DMARC) live in **separate namespaces within the same zone**. Mail servers never read your A record to deliver mail; browsers never read your MX to load the site. So:

- You can add a full email stack to a domain with a live site at **zero risk to the site**.
- The one caveat: *add* records — a whole-zone replace (the PUT trap in [Domains and DNS](domains-and-dns.md)) can drop the A/CNAME along the way. Add at the narrowest scope and the site cannot be touched.

Write the records at whoever `dig NS` names as the DNS host — not necessarily the registrar.

## The record set

| Record | Where | What it does |
|---|---|---|
| **MX** | apex (or subdomain) | Where inbound mail is delivered — defines your mail provider. |
| **SPF** | `TXT @` → `v=spf1 include:... ~all` | Which servers may send mail claiming to be your domain. **Exactly one** `v=spf1` record per name — a second one is a permanent error, not a merge. |
| **DKIM** | `TXT`/`CNAME` at `selector._domainkey` | Public key(s) that verify your provider's cryptographic signature on each message. |
| **DMARC** | `TXT _dmarc` → `v=DMARC1; p=...` | Policy tying SPF/DKIM to the visible `From:` domain (alignment), plus reporting. |
| **Custom MAIL FROM** | `MX` + SPF `TXT` on a bounce subdomain | Makes SPF *align* with your domain instead of the ESP's (below). |
| **BIMI** | `TXT default._bimi` | Logo-in-inbox. Real constraints — see below. |

### Aligned MAIL FROM: the piece most setups miss

SPF is checked against the invisible envelope sender (MAIL FROM), which ESPs default to *their own* domain — so SPF can pass yet not **align** with your `From:` address, leaving DMARC to ride on DKIM alone. The fix (shipped and live-verified on customdomain.ai, 2026-07): configure a custom MAIL FROM subdomain at your ESP (e.g. `bounce.yourdomain.com`), publish its two records — an MX pointing at the ESP's feedback host and its own `v=spf1` TXT — and SPF then passes *and aligns*. Your DMARC no longer has a single point of failure.

### DMARC: progressive enforcement, not day-one `p=reject`

The policy we ship and recommend:

1. **Start**: `v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com` — monitoring only. Make `dmarc@` a real mailbox you can read.
2. **Watch 2–4 weeks** of aggregate (rua) reports — parse the raw XML with a tool like `parsedmarc` rather than eyeballing it. You're looking for legitimate senders you forgot (CRM, billing, newsletter tools) failing alignment.
3. **Tighten**: `p=quarantine; sp=quarantine; pct=100; adkim=r; aspf=r` once reports are clean.
4. **Finish**: `p=reject` after another clean window.

Two details from the RFC that trip people (documented behavior, RFC 7489): forensic-report options like `fo=1` are **inert without a `ruf=` address** — set both or neither; and `sp=` covers subdomains, which spoofers otherwise use to ride your clean apex reputation.

### BIMI: publish it, expect little (as of 2026-07)

Verified expectations, not vendor marketing: a **self-asserted** BIMI (SVG only, no certificate) realistically displays **only in Fastmail** today. Yahoo/AOL gate display on bulk-sender volume and reputation; Gmail and Apple require a Verified Mark Certificate (VMC/CMC), which requires a registered trademark. The SVG must be the Tiny-PS profile *and* Gmail-ready (absolute pixel size ≥96, `<title>` present). Publish it as cheap future-proofing after DMARC is at enforcement — just don't budget real effort against it.

## Architecture: match email to the infrastructure you have

Pick the design the available infrastructure actually supports, and say so honestly when a goal isn't reachable:

| Situation | Right architecture |
|---|---|
| Your platform/host offers native mailboxes | Use them — one vendor, one DNS block, simplest. |
| You only need to **receive** at the domain | Forwarding to an existing inbox (MX → a forwarder). Cheap — but **sending as** the domain still requires adding it to a sending service and publishing that service's SPF/DKIM. Receiving and sending are separate capabilities. |
| Real mailboxes + calendar + sending | Google Workspace or Microsoft 365: their MX + SPF include + DKIM records at your DNS host. |
| Transactional/bulk sending (app or campaigns) | A dedicated **subdomain** with its own SPF/DKIM/DMARC (below). |

Two subdomain-isolation patterns we run in production:

- **Marketing off the apex**: send campaigns from a dedicated subdomain (e.g. `mail.` or `news.`) so a campaign mistake burns the subdomain's reputation, not your primary domain's. Cheapest to do *before* the first send, when everything has zero reputation anyway.
- **Agent mail segregated**: an AI agent that sends email gets its own subdomain (e.g. `ai.yourdomain.com`) with full SPF/DKIM and DMARC `p=reject` — its reputation and its failure modes are firewalled from the humans' domain (shipped 2026-07).

Know the limits of isolation: with a shared sending account (e.g. one SES account), reputation, suppression lists, and quotas are **account-wide** — per-domain separation isolates metrics, not the account-level pause a bad blast can trigger.

## The sender mandates (why none of this is optional)

Adversarially re-verified 2026-07 (41 claims confirmed, 5 refuted, in that research pass): Gmail's and Microsoft's bulk-sender mandates are real and enforced — SPF **and** DKIM, a DMARC record (minimum `p=none`), aligned `From:`, one-click List-Unsubscribe on bulk mail, and spam-complaint rates under ~0.3%. Unauthenticated mail to these providers is now rejected or junked outright, at any volume.

And the honest counterweight: **authentication is necessary, not sufficient.** A brand-new domain with a perfect 10/10 auth setup still gets graymailed by Outlook for roughly 2–6 weeks on reputation alone (community-reported, consistent with our experience). The remedy is consistent modest volume and time — not more DNS records.

## Verify it: the write-side check

After setup, prove it end to end:

1. **mail-tester.com** — send a real message to the throwaway address it gives you; it scores SPF/DKIM/DMARC/alignment/content. Gotcha we hit: a one-line test body scored 8.9 because of an image-to-text-ratio heuristic; the same setup with a realistic body scored **10/10**. Test with real content.
2. **Check headers on a received message** (Gmail: "Show original") — you want `spf=pass`, `dkim=pass`, `dmarc=pass`, and the SPF domain matching your MAIL FROM subdomain (alignment).
3. **Read your first rua reports** in ~48h — they list every source sending as your domain, including the ones you forgot and the ones that aren't you.

## Audit any domain in 60 seconds: the read-side check

The same records make any domain's posture publicly auditable — run this on your own domain quarterly, and on any domain you're about to depend on:

```bash
dig +short MX yourdomain.com                       # who handles mail
dig +short TXT yourdomain.com | grep spf1          # SPF (count the v=spf1 lines: must be 1)
dig +short TXT _dmarc.yourdomain.com               # DMARC policy
dig +short CNAME autodiscover.yourdomain.com       # M365 fingerprint
dig +short TXT  google._domainkey.yourdomain.com   # Workspace DKIM fingerprint
dig +short CNAME selector1._domainkey.yourdomain.com  # M365 DKIM fingerprint
```

Reading the results:

| Observation | Meaning |
|---|---|
| MX → `*.protection.outlook.com` | Microsoft 365 tenant |
| MX → `*.google.com` / `googlemail.com` | Google Workspace |
| MX → mimecast / proofpoint | Enterprise security gateway in front |
| SPF `include:` entries | Every third-party service authorized to send as the domain — audit yours; each is an attack/reputation surface |
| No `_dmarc` record, or `p=none` forever | Domain is spoofable; posture neglected |
| DMARC `rua=` pointing at a third party | Who's monitoring their mail |

## Gotchas

1. **Two `v=spf1` records.** Only one is allowed per name; a second causes `permerror` (SPF effectively off). Merge the `include:` mechanisms into one record.
2. **The 10-DNS-lookup SPF limit.** Each `include:` costs lookups; past 10, SPF permerrors. Flatten or prune rarely-used services.
3. **MX without DKIM/DMARC.** Receiving works with MX alone, which fools people into thinking they're "done" — sending trustworthily needs the full set, aligned.
4. **Mail identity verification dies during domain migrations.** ESP domain identities are verified against DNS; if the zone moves and records lapse (or NXDOMAIN during a transition), the identity fails and sending stops. Re-check identity status as part of every cutover — one migration found an identity that had been silently FAILED for two months ([Domain migrations](domain-migration.md)).
5. **Link-domain mismatch.** After a cutover, apps kept minting tracked links and unsubscribe URLs on the *old* base URL — authenticated mail full of off-brand links is both a deliverability and a trust problem. Check the link base, not just the `From:`.
6. **The apex MX that isn't yours.** Zones accumulate history — one we automated against carried the owner's live personal-inbox MX. Read–merge–upsert; never replace a zone wholesale.
7. **Consumer Gmail can't be configured to send-as an external domain via API** — the send-as flow with an SMTP relay is Workspace-only (domain-wide delegation); for a personal Gmail the settings UI is the only lane (verified 2026-07).
8. **Reports nobody reads.** `rua=` pointing at a dead mailbox is monitoring theater. Wire it to a real inbox and a parser, or you'll learn about spoofing from your customers.

## Related

- [Domains and DNS](domains-and-dns.md) — where these records get written, and the registrar-vs-DNS-host split
- [Domain migrations](domain-migration.md) — preserving the email zone through a cutover
- [Entities, E-E-A-T, and trust](../foundations/entities-and-trust.md) — the wider trust surface this posture feeds
- [Off-site signals](../ai-search/offsite-signals.md) — the outreach and review flows that depend on deliverability
- Source skills: [custom-domain-email-dns-diagnosis](https://github.com/ever-just/agentskills/tree/main/skills/custom-domain-email-dns-diagnosis), [domain-email-enumeration](https://github.com/ever-just/agentskills/tree/main/skills/domain-email-enumeration)
