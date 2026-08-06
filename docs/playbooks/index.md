# Playbooks

The rest of this book teaches one surface at a time. The playbooks put the chapters in **execution order for a specific situation** — what to do first, what gates what, how long each step honestly takes, and how to prove it worked before moving on. Every playbook here is distilled from a program we actually ran, failures included; nothing is a hypothetical sequence.

## Which playbook

| You are… | Run this | Time to first result |
|---|---|---|
| Launching (or relaunching) a SaaS / dev-tool product | [Launch a SaaS product](saas-launch.md) | Baseline + crawlability in an afternoon; full program over 2–4 weeks |
| Running a local service business | [Launch a local business](local-business.md) | The invisibility check in 30 minutes; full stack over 2–4 weeks |
| Not sure anything is wrong — or sure something is | [The 30-minute AI visibility audit](ai-visibility-30min.md) | 30 minutes, today |
| Done launching, now keeping it alive | [The operating cadence](operating-cadence.md) | 15–30 min/week once set up |

If neither launch playbook matches your situation, [Choose your path](../start/choose-your-path.md) maps six more scenarios to chapter sequences, and the [master checklist](../start/master-checklist.md) is the surface-agnostic version of everything below.

## How the playbooks are built

Each playbook is a sequence of **numbered phases**. Every phase has the same anatomy:

- **Goal** — what is true when the phase is done.
- **Actions** — a checklist, in order.
- **Effort** — an honest range for one operator. Where the source program used agent fleets to compress days into hours, we say so and give the human-scale number.
- **Verification** — how to prove it worked *at the layer that serves users* (the rendered page, the public robots.txt, the live SERP — never the CMS field or the deploy log).
- **Deep dives** — links into the chapters that explain the mechanics.

Effort labels used throughout:

| Label | Means |
|---|---|
| **Minutes** | under 30 minutes |
| **Hours** | 1–4 focused hours |
| **Day+** | one to several working days |
| **Ongoing** | recurring — belongs to the [operating cadence](operating-cadence.md) |

## The shared spine

Both launch playbooks follow the same skeleton, because the dependency order is the same everywhere:

1. **Measure before you touch anything.** Record Search Console / Bing baselines first — otherwise you will never know what worked. ([Measurement](../foundations/measurement.md))
2. **Unblock the gate.** Crawlability failures (WAF challenges, `noindex` traps, robots mistakes) zero out everything downstream. Both case studies behind these playbooks began with a site that was **invisible** — one behind a bot-challenge wall, one silently `noindex`-ed by its own CMS.
3. **Establish identity.** Structured data and entity signals so machines resolve one confident thing. ([Structured data](../google/structured-data.md), [Entities and trust](../foundations/entities-and-trust.md))
4. **Win winnable queries.** Keyword reality check, then answer-first content for intent queries. ([Keyword strategy](../google/keyword-strategy.md), [Content that gets cited](../ai-search/content-that-gets-cited.md))
5. **Amplify where your audience's machines look.** MCP registry and GitHub for agents; GBP, reviews, and citations for local; off-site presence for answer engines.
6. **Operate.** Automation for the hourly loops, a human rhythm for the rest. ([The operating cadence](operating-cadence.md))

## Rules that hold in every playbook

- **Baseline first, always.** A change made before the baseline is recorded is a change you can never evaluate.
- **Verify at the serving layer.** The recurring failure mode in the source programs was "verified" work that never rendered — a DB row that wasn't the live page, a robots.txt correct on the internal host and stale on the public one. Fetch what crawlers fetch.
- **Honest content only.** Fabricated reviews, invented stats, and stock photos captioned as real work are discoverability *risks*, not shortcuts — schema policies and FTC rules both punish them. ([Authenticity audits](../local/authenticity.md))
- **Log append-only.** Every phase ends with a dated log entry. Corrections get appended, never rewritten — see [what to log](operating-cadence.md#what-to-log-and-where).
- **Unmeasured means unmeasured.** Where a source program didn't measure an outcome, the playbook says so instead of implying success.

## Related

- [The master checklist](../start/master-checklist.md) — the same ground as one flat list
- [Choose your path](../start/choose-your-path.md) — more scenarios, mapped to chapters
- [How discovery works in 2026](../start/how-discovery-works.md) — why the spine is ordered this way
- [Case studies](../case-studies/index.md) — the programs these playbooks are distilled from
