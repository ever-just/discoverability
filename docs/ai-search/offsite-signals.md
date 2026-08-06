# Off-site signals

**Roughly 85–93% of the citations AI answer engines produce point at somewhere other than the brand's own website.** Reddit alone is around 40% of all AI citations (~47% for Perplexity); G2 and Capterra dominate B2B software answers; Yelp, BBB, and the local aggregators dominate local ones. Which means once your own site is clean, the ceiling on your AI visibility is set by what *other* people's pages say about you. This chapter is the map of those sources, how to earn presence on each without astroturfing, and the honest limit on how much of it an agent can do for you.

(All percentages: external research, compiled 2026-07. Treat them as direction and magnitude, not constants.)

## Where each engine sources from

Engines do not share a citation pool — only about **11% of domains cited by ChatGPT are also cited by Perplexity**. Optimizing for one is not optimizing for all.

| Engine | Skews toward | Practical consequence |
|---|---|---|
| **ChatGPT Search** | Bing's top organic results (~87% overlap), encyclopedic sources, Yelp/BBB for local | Win Bing, and win the directories Bing ranks — [Bing & Beyond](../bing/index.md) |
| **Perplexity** | Reddit (~47% of its citations), and content less than ~30 days old (recrawls fresh pages in ~2–7 days) | Community presence plus publishing cadence; seasonal content shipped *ahead* of season |
| **Google AI Overviews / AI Mode** | Google's index, structured data, Business Profile, YouTube and other multimodal sources | Schema + GBP + video, not just text pages — [Google](../google/index.md) |
| **Claude** | Live search plus training-corpus knowledge | Durable, widely-referenced sources (docs, GitHub, established directories) |

Two structural facts follow. First, **third-party presence is the highest-ceiling lever and the slowest** — the [lever list](geo-fundamentals.md#the-prioritized-lever-list) puts it at #5 for exactly that reason: do the cheap gates first, then start this, because it compounds over months. Second, **you cannot fake it**. Engines cite forums *because* forums are authentic; the moment your presence there reads as manufactured, it gets removed by moderators, and the reputational damage outlives the citations you were chasing.

## The five source classes

### 1. Review platforms — the confidence threshold

For AI, reviews are not a ranking factor so much as a **confidence gate**. The research pattern (external, compiled 2026-07): assistants recommending a local business skew heavily toward those above roughly a **4.3-star** average with a non-trivial review count; reviews carry roughly 20% of local-pack weight; and response rate itself is a signal — responding to 80%+ of reviews is the operational bar.

What to do:

- **Be on the right two or three platforms**, not all of them. B2B software: G2 and Capterra, in the correct category. Local services: Google Business Profile first, then Yelp and BBB. Everything else is a rounding error.
- **Ask for reviews systematically and legally** — a post-job or post-onboarding request email, a direct write-a-review deep link on your site and in the email. On one case-study property, three separate acquisition paths (site CTA, homepage CTA, automated post-job email) were wired at once (measured, 2026-07).
- **Except on Yelp.** Yelp's policy prohibits soliciting reviews and demotes businesses that do. Read each platform's rule before you automate anything.
- **Never buy, invent, or incentivize reviews.** Beyond the platform bans, marking up fabricated reviews as schema is a Google policy violation with manual-action risk → [Reviews — real ones only](../local/reviews.md).
- **Display them honestly.** If your site shows a rating, sync it from the source rather than hardcoding — hardcoded social proof drifts within days. We watched a review count diverge across a single site (47 vs 48 vs 51 on different pages) inside a week before a single-source-of-truth sync fixed it (measured, 2026-07).

### 2. Communities — Reddit, HN, and the niche forums

The single biggest citation source, and the one most likely to blow up in your face.

- **Find the threads engines already cite.** Run your intent queries through Perplexity and note which Reddit/HN/Stack Overflow/DEV threads appear as sources. Those exact threads — plus the subreddits and tags around them — are your target list.
- **Answer as a practitioner, disclose as a vendor.** The pattern that works: contribute a genuinely useful, specific answer to a question you actually know; mention your product only when it's the honest answer; disclose the affiliation in the same breath. Communities tolerate a vendor who is useful and transparent. They destroy one who isn't.
- **Local businesses have local equivalents** — neighborhood Facebook groups, Nextdoor, city subreddits, trade subreddits. Same rules.
- **What "without astroturfing" means concretely:** no sockpuppets, no paid or incentivized posts pretending to be organic, no seeded "which tool should I use?" questions answered by your own second account, no undisclosed employees posting as customers. All of these are detectable, all get deleted, and all are the exact behavior the [honesty doctrine](../local/authenticity.md) exists to prevent. The doctrine applies to us too: this book won't teach a shortcut it wouldn't run on its own properties.

### 3. Directories and citations — the NAP program

For local businesses this is the highest-ROI off-site work there is. The research finding (external, compiled 2026-07): businesses whose **name, address, and phone are byte-identical across roughly 20 directories are about 3x more likely to appear in AI local recommendations.** The mechanism is unglamorous — assistants pull contact facts from a narrow set of trusted aggregators, so a wrong number on one of them means the AI confidently hands your customer a dead line.

The program shape, from a 50-platform build-out we ran (2026-07-30):

| Step | What happens | Who does it |
|---|---|---|
| 1. Fix the canonical record first | One business name, one phone, one address decision, one hours set, one category — settled *before* any listing is created | Human decision |
| 2. Research the platform universe | Profile each candidate: signup requirements, whether an address must be public, category taxonomy, character limits, real value | Agent |
| 3. Triage | Sort into worth-doing / executable-but-worthless / excluded (paid-only, dead, or requiring you to publish a home address) | Agent, human confirms |
| 4. Build the content library | Descriptions pre-written to each platform's exact character limits, category mappings per platform, canonical NAP block | Agent |
| 5. Claim vs create audit | Search each directory by name and by phone — is there an existing (possibly wrong) listing to claim, or a clean slate? | Agent |
| 6. Create the accounts and submit | Signup, passwords, SSO, CAPTCHAs, phone/postcard verification, payments | **Human only** |
| 7. NAP audit sweep | Re-fetch every live listing and diff every field against the canonical block | Agent |

Our triage of ~50 platforms came out at **20 worth doing, 12 executable but worthless, 17 excluded** — which is the real lesson: the "submit to 500 directories" services are selling you the worthless 12 and the excluded 17. Specific findings from that pass (as of 2026-07, all subject to change):

- Several legacy directories share a single login across their family of sites — one signup covers three listings.
- Some directories require typing a street address but offer a genuine "don't display" toggle; some don't. If you're home-based, that distinction decides whether the platform is usable at all.
- At least one signup's consent text authorizes automated marketing calls that override a Do-Not-Call registration. Read the consent text; it's part of the cost.
- Apple's business listing program was reorganized in **April 2026** into full organization enrollment (tax ID, DNS TXT verification, a never-before-used email, and a deletion clock on incomplete applications) — every guide written before that date is wrong.
- Bing Places' real value in 2026 is **Copilot and ChatGPT grounding**, not Bing traffic. Judge it on that.

### 4. Comparison and roundup content

"Best X" listicles and "X alternatives" pages are disproportionately cited because they match the shape of the question. You have three moves, in increasing order of effort:

1. **Get included in existing roundups.** Find the ones that rank and get cited, then contact the author with a factual, checkable pitch. Some are pay-to-play; those are usually the ones AI doesn't cite.
2. **Publish your own comparison page** — honest, table-shaped, and willing to say where you lose. See the table-extraction advantage in [Content that gets cited](content-that-gets-cited.md).
3. **Create the roundup that doesn't exist.** When we found no curated list for a product category, we created one — listing competitors honestly alongside our own entry (2026-07). A genuinely useful list earns citations; a list with one entry earns contempt.

### 5. GitHub, package registries, and ecosystem repos

For developer-facing products this class is both a citation source and an agent-discovery surface, and it's covered in depth in [GitHub as a discovery surface](../agents/github-as-discovery.md). The headline facts: GitHub's repo search indexes only **name, description, and topics** (README text requires an `in:readme` qualifier), topics are exact-match and many category topics are unclaimed, and permissively-licensed repos enter model training corpora. Adjacent to it: package-registry listings, ecosystem template repositories, and awesome-lists — each of which puts you inside someone else's discovery flow. The [MCP Registry](../agents/mcp-registry.md) is the same play for agent tooling.

## Entity corroboration underneath all of it

Off-site mentions only compound if engines can tell they're all about *the same* organization. That's the entity layer: an `Organization` node with a `sameAs` array pointing at every profile you own, a Wikidata item if you're notable enough, and byte-identical name, logo, and description across LinkedIn, GitHub, Crunchbase, and the directories. A single-URL `sameAs` gives an engine essentially no corroboration graph — a real audit finding on our own SaaS property (2026-07-16). Full treatment: [Entities, E-E-A-T, and trust](../foundations/entities-and-trust.md).

## The honest limit: what an agent cannot finish

If you're running this program with an AI agent, know where it stops. An agent can research every platform, triage them, pre-write every description to every character limit, map categories, audit existing listings, fill forms, and verify results afterward. It **cannot create accounts** — signup flows mean email verification, password creation, SSO, CAPTCHAs, phone or postcard verification, and sometimes payment. Our 50-platform program ran to completion on everything else and then stopped dead exactly there (measured, 2026-07-30).

Plan for it: the agent's deliverable is a **human queue** — a tab listing each platform, the exact values to paste, and the verification step — not a finished citation profile. Budget a human afternoon.

Three prerequisites that must be resolved by a human *before* the first listing is created, because fixing them afterward means editing every listing:

1. **One canonical phone number.** If a second number circulates on printed collateral or old profiles, decide which one wins first.
2. **The address decision.** Home-based businesses must decide what is publishable; that decision eliminates a chunk of the platform list.
3. **No false badge or accreditation claims.** On one case-study property we found a badge asserting accreditation while the very organization's own linked profile said otherwise — a deceptive-advertising exposure under FTC rules, and exactly the sort of claim a directory build-out would have replicated across twenty sites. The true version of the claim (a rating, without accreditation) was available and worked fine.

## How you know it worked

Off-site work has a slow, indirect feedback loop, so measure at three levels:

1. **Listings exist and agree.** Fetch every live listing and diff name/address/phone/hours/URL against your canonical block. Zero diffs, or a dated fix ticket per diff.
2. **Review metrics moved.** Rating above the ~4.3 confidence bar, review count trending up, response rate above 80%.
3. **The citations changed.** Re-run your [query battery](ai-visibility-audit.md) and look at *who gets cited* — not just whether you're mentioned. When a directory, review page, or forum thread that mentions you starts appearing as a source, that's this work landing. Log it in the audit sheet with the date; it's the only way to attribute anything here.

Expect months, not weeks. The compounding is real but it is not fast.

## Gotchas

- **Astroturfing.** Fake reviews, sockpuppet threads, undisclosed employee posts. Detectable, removable, and it destroys the exact authenticity that made the source citable.
- **"Submit to 500 directories" services.** They sell volume from the worthless tail. Twenty correct, consistent listings beat five hundred inconsistent ones — and inconsistency is actively harmful, because a wrong phone number in a trusted aggregator becomes a wrong phone number in an AI answer.
- **Creating listings before the canonical record is settled.** Changing a phone number across twenty directories is a multi-week chore with verification steps on each one.
- **Ignoring per-platform policy.** Yelp's don't-ask rule, consent language that signs you up for marketing calls, address-display requirements. Read before you register.
- **Optimizing for one engine's sources.** ChatGPT and Perplexity overlap on only ~11% of domains — a Reddit-only strategy leaves ChatGPT untouched and vice versa.
- **Treating off-site as a substitute for on-site.** It's the highest ceiling, not the first move. If AI bots can't fetch your site, third-party mentions send traffic to a page nothing can read ([AI crawlers](ai-crawlers.md)).
- **Letting listings rot.** Hours change, phone numbers change, services change. An annual NAP sweep belongs in the [operating cadence](../playbooks/operating-cadence.md).

## Related

- [GEO fundamentals](geo-fundamentals.md) — where this lever sits in the priority order and why
- [Auditing your AI visibility](ai-visibility-audit.md) — the measurement loop that tells you this is working
- [Reviews — real ones only](../local/reviews.md) — the schema and policy side of review platforms
- [Authenticity audits](../local/authenticity.md) — the doctrine that rules out the shortcuts
- [Entities, E-E-A-T, and trust](../foundations/entities-and-trust.md) — making all these mentions resolve to one entity
- [GitHub as a discovery surface](../agents/github-as-discovery.md) — the developer-audience version of this chapter
- [Google Business Profile](../google/business-profile.md) — the single most important local listing
- Source skills: [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization), [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema), [marketing-site-authenticity-audit](https://github.com/ever-just/agentskills/tree/main/skills/marketing-site-authenticity-audit)
