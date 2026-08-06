# Case studies

Every framework in this book was extracted from real work on four properties between June and August 2026. These pages are the dated records of that work: what shipped, what broke, what the numbers actually said, and what remains unmeasured. If the rest of the book tells you *what to do*, this part shows the frameworks operating under real constraints — platform bugs, owner decisions, Google's actual behavior, and the limits of agent automation.

## How to read these

Each case study follows the same shape:

- **"What this proves"** — the claims this case is evidence for, up front.
- **Timeline** — dated, chronological entries. Incidents are marked with warning boxes; they are the most valuable parts.
- **The numbers** — only real, recorded figures: baselines captured before changes, counts, validation results. Anything not measured is explicitly marked **unmeasured** — this book does not round up.
- **What worked / What failed / What we'd do differently** — the honest ledger.
- **Chapters this case feeds** — where each lesson became a reusable chapter.

Three doctrines govern these pages:

1. **Honesty.** The book preaches authenticity as a ranking strategy ([authenticity audits](../local/authenticity.md)), so it applies the same standard to itself: failures are documented with root causes, and outcomes that postdate the record are not claimed.
2. **Append, never rewrite.** Each case is a point-in-time record. Later developments get appended with dates; the original narrative stays as written.
3. **Pattern-level detail.** No credentials, no customer data, no internal infrastructure specifics. Implementation detail appears only where it is already public (largely via [ever-just/agentskills](https://github.com/ever-just/agentskills)) or where the pattern itself is the lesson.

## The four cases

| Case | Type | Window | Core lesson |
|---|---|---|---|
| [customdomain.ai](customdomain-ai.md) | SaaS product (human buyers + AI agents) | 2026-07-01 → 07-21 | Intent and agent discovery must be engineered on purpose — and verified at the layer that actually serves users, because the differentiated path can be quietly broken while everything looks shipped |
| [Heads Up Outdoor Services](headsup.md) | Local service business | 2026-07-09 → 07-30 | The two catastrophic problems were infrastructure defaults invisible from inside the CMS; the durable local strategy is verifiable honesty |
| [everjust.app tenants](everjust-tenants.md) | Multi-tenant CMS platform | 2026-07-02 → 08-03 | A Host-rewriting reverse proxy makes a CMS misread its own identity — robots, noindex, sitemaps, and canonicals then fail as a *class*, hidden behind layered caches |
| [brogav.com](brogav.md) | Third-party traffic estimation | 2026-06-12 → 06-13 | Free, open sources triangulate a defensible traffic estimate — if every number carries a confidence tier and micro-traffic findings change the strategy, not just the report |

## What each one proves

**customdomain.ai** is the book's origin story for the [branded-vs-intent reframe](../ai-search/geo-fundamentals.md) and its extension to a second audience: AI agents. It proves that a product can have every discovery surface formally shipped — schema, registry listing, OAuth discovery — while the agent *usability* path is broken in ways only an adversarial audit finds. It also supplies the [SERP tokenization crisis](../google/keyword-strategy.md) (a brand name Google parses as two generic words), the maximal [schema @graph](../google/rich-results.md) worked example, and the [GitHub-as-discovery](../agents/github-as-discovery.md) funnel.

**Heads Up Outdoor Services** is the local stack end to end: [LocalBusiness schema](../local/local-business-schema.md), [policy-safe reviews](../local/reviews.md), [service-area pages that survive scrutiny](../local/service-areas.md), and [authenticity remediation](../local/authenticity.md). Its two headline incidents — a WAF challenge that erased the domain from search, and a sitewide noindex that shipped for 11 days — are why this book insists you [test the public surface the way crawlers see it](../technical/rendering-and-waf.md).

**everjust.app tenants** is the platform view of the same period: the [reverse-proxy trap catalog](../technical/reverse-proxy-cms.md) discovered one incident at a time, the [domain-cutover method](../technical/domain-migration.md) that preserves email zones, and [IndexNow](../bing/indexnow.md) as a platform feature. Read it if you run *any* site behind a proxy that rewrites the Host header.

**brogav.com** is the smallest case and the purest method: estimating a micro-traffic site's reality from free sources with explicit [confidence tiers](../foundations/measurement.md) — and what an honest ~1,200-visits-a-month answer means for strategy.

## Related

- [How discovery works in 2026](../start/how-discovery-works.md) — the model these cases validated
- [Measurement and baselines](../foundations/measurement.md) — why every case records numbers *before* changes
- [Playbooks](../playbooks/index.md) — the same sequences, generalized into runnable form
- [Skill index](../appendix/skills-index.md) — the reusable skills distilled from this work
