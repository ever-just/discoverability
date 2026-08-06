# Google Business Profile

**For a local business, the Business Profile — not the website — is the primary discovery surface, and it can be live yet completely invisible.** This chapter covers claiming and completing a profile, how categories and attributes decide which searches you can appear for, the API-access reality (the Business Profile APIs ship at **effective quota zero** until Google approves an access application, so apply early), how the profile ties back to your site's schema so machines resolve one entity, and the diagnosis procedure for the failure mode we hit in production: a real business whose profile returned nothing in Google's own Places search while its competitors resolved instantly.

## Why the profile outweighs your website

For local queries, Google answers from the profile, not the page. So do the answer engines: research the local case study relied on (industry studies, as of 2026-07 — **reported**, not our measurement) put it bluntly — Google Business Profile, reviews, and trusted directories outweigh the marketing site for local AI recommendations, AI assistants apply a review-based confidence threshold (around a 4.3-star average) before recommending anyone, and Google's AI Overviews skew toward schema + GBP for local intent.

| Surface | What decides it |
|---|---|
| Map pack (the 3-result block) | Profile completeness, primary category, proximity, reviews, NAP consistency |
| Google Maps | The profile, entirely |
| Local knowledge panel | The profile, corroborated by your site's entity graph |
| AI local recommendations | Profile + reviews + a narrow set of trusted directories (Yelp, BBB, aggregators) — reported |
| Organic blue links for local queries | Your website (the rest of this part) |

The practical consequence: if you have one hour and a local business, spend it on the profile. If your profile is unverified, **nothing on your website can compensate**.

## Claim and verify

1. **Search for the business first** in Google Maps and in Google Search. One of three states: no listing (create one), an unclaimed listing Google generated (claim it), or a listing someone else manages (request ownership — this runs a transfer process with a waiting period).
2. **Claim it** from the profile's "Own this business?" link, signed in as the *business* account — not whatever personal Google account happens to be in the browser. Business identity is an access-control decision you live with for years; the same hygiene rule applies here as to [Search Console properties](search-console.md).
3. **Verify.** Google picks the method (video, phone, mail, email, or Search Console association) based on category and risk signals; you often don't get to choose. Video verification has become common for service-area businesses and can take days to weeks including re-submissions.
4. **Budget lead time.** Verification is the long pole in a local launch and it is *not* under your control. Start it on day one, in parallel with website work, not after.

**How you know it worked:** the profile shows "Verified" in the dashboard, and — the check that actually matters — searching the business name in Maps returns it. Passing step one is not passing step four; see [the invisible-profile failure mode](#the-invisible-profile-failure-mode).

## Categories, services, and attributes

**The primary category is the highest-leverage field on the profile** (documented Google behavior): it decides which query classes you are eligible to appear for at all. Everything else refines.

| Field | Rule |
|---|---|
| **Primary category** | Exactly one, from Google's fixed taxonomy. Pick the category that matches the query you most want to win, not the most flattering label. If you're a lawn-care company, "Lawn care service" beats "Landscaper" if lawn care is the money service. |
| **Additional categories** | Up to nine more (documented). Add only categories you genuinely serve — each one is a claim, and irrelevant categories dilute rather than expand. |
| **Services** | Free-text service items grouped under your categories, each with an optional description and price. This is where long-tail service vocabulary lives; mirror the service names used on your [service pages](../local/index.md). |
| **Service areas** | For businesses that travel to customers. Google caps how many you can declare (the profile UI shows the current limit). Declare only places you actually work — the same evidence standard as [service-area pages](../local/service-areas.md). |
| **Attributes** | Category-specific (wheelchair accessible, veteran-owned, online estimates, etc.). Some are owner-set; others Google derives from reviews and cannot be edited directly. Fill every owner-set attribute that is true. |
| **Hours** | Regular plus special/holiday hours. Wrong hours are a trust defect a competitor's correct hours beat. |
| **Description** | 750 characters (documented limit). Front-load what you do and where. Write it answer-first, like the [content that gets cited](../ai-search/content-that-gets-cited.md) — assistants quote it. |
| **Photos** | Real photos of real work, with the same [authenticity standard](../local/authenticity.md) as the site. Stock photography on a local profile is a credibility risk, not a filler. |
| **Products / posts** | Low-effort freshness signals. Posts expire; treat them as a cadence item, not a launch item. |

**Consistency is the whole point.** Name, address, and phone must be byte-identical across the profile, your website, and every directory. AI assistants pull contact details from a narrow trusted set, so a wrong number in one aggregator becomes a wrong number in an AI answer (reported). The directory build-out itself is covered in [off-site signals](../ai-search/offsite-signals.md).

## The API reality: quota starts at zero

If you plan to automate anything — pulling reviews, syncing hours, updating multiple locations — **read this before you architect it.** Verified in production, 2026-07:

- **The Business Profile APIs are OAuth-only.** An API key never works. (Same class of failure as [Search Console](search-console.md#the-fully-api-driven-route-service-account): a Maps key is not a Business Profile credential.)
- **Enabling the API in your cloud project does not grant quota.** A freshly enabled project has *effective quota zero* until Google approves a separate access application. What you see in that state: `GET .../accounts` returns **429 `RESOURCE_EXHAUSTED` with an empty quota bucket**. That is the *pending-approval* signal, not rate limiting. **Do not retry, do not add backoff, do not build on it** — we lost time treating it as a transient error.
- **Approval takes real lead time.** The application asks for business justification and is reviewed by humans. In our engagement the quota was still pending when the work window closed. **Apply on day one** if automation is anywhere in the plan; the browser lane covers you meanwhile.
- **The practical lane for profile edits is a signed-in browser session.** That is not a workaround, it's the supported path for a single business. Automate it only to the extent of preparing the exact content to paste.

### What you can get without approval

The **Places API** is a different, ungated surface and answers a different question. It returns the listing's identity, current rating, and total rating count — enough to keep your site's displayed rating honest — but it **hard-caps at 5 reviews per pull, on any key and any API version** (verified across two keys and both the legacy and v1 endpoints). Accumulating pulls over time cannot backfill older reviews; the only official route to *all* of a business's reviews is the approval-gated Business Profile API. Design your [review pipeline](../local/reviews.md) around that constraint instead of discovering it late.

One version gotcha: a listing ID can be effectively version-locked. We hit a `place_id` that the legacy Place Details endpoint rejected outright while the v1 endpoint resolved it fine. Use Places v1 and stop maintaining legacy calls.

### Keyless credentials, at concept level

Where organization policy forbids downloadable service-account keys — a common and reasonable posture — the pattern is **service-account impersonation**: a human or business account that holds the token-creator role on the service account mints short-lived access tokens for it on demand. Nothing long-lived is ever stored on disk. Two things to know before you debug it: the token-creator grant needs a minute or two of IAM propagation before it works, and your CLI's displayed "active account" can differ from the account actually configured — trust the configured value. Whichever route you use, a credential file that must exist belongs outside the repo with restricted permissions, in a dedicated project.

## Reviews are the local ranking currency

Reviews are simultaneously a ranking input, a conversion lever, and the confidence threshold AI assistants apply before recommending you. Three mechanics belong on the profile side:

- **Build acquisition paths, not campaigns.** What we shipped: a "leave us a review" deep link (Google's `writereview?placeid=` URL) on the reviews page *and* the homepage, plus an automated review-request email sent a set interval after a job closes. Three paths beat one ask.
- **Watch the CTA labels.** A caught bug from that build: the "Read all reviews on Google" link actually pointed at the *write-review* form. A mislabeled review CTA converts nobody and looks like solicitation.
- **Respond to reviews.** Response rate is treated as a signal, and public responses are content AI reads (reported: respond to >80%).

Never fabricate, paraphrase, or selectively edit a review, and never put an `aggregateRating` for those reviews on your site's business node — that's a policy trap with manual-action risk. The full policy, the Service-node placement rule, and the live-sync architecture are in [Reviews — real ones only](../local/reviews.md).

## Connect the profile to your site's schema

The goal is that Google, Bing, and answer engines resolve **one entity** from two sources rather than two similar-looking businesses. The joins that do that work, all shipped on the local case study:

- **`hasMap`** on the sitewide `LocalBusiness` node, pointing at the Maps listing URL, plus the listing's CID-form Maps link. This is the most direct site→profile pointer available in schema.org.
- **`sameAs`** listing the profile and the same social/directory profiles the GBP links to — Facebook, Instagram, Yelp, BBB. A single `sameAs` is weak corroboration; a consistent set is an entity.
- **NAP byte-identical** between the schema node and the profile: same legal name string, same address formatting, same phone format.
- **`areaServed`** as `City` objects matching the profile's declared service areas — not a superset. If the site claims more cities than the profile, you've created a contradiction machines can see.
- **No `aggregateRating` on the business node.** Stars belong natively on the profile; on your site they belong on a `Service` node backed by visible reviews ([why](rich-results.md#review-stars-and-the-self-serving-rule)).

Two adjacent rules: if you display Google's brand mark next to your rating, use the official four-color "G" **unaltered** — Google's identity guidelines require the mark as published, so no hand-drawn or recolored versions. And Google-measured drive times are Maps Content that cannot be baked into your page copy; publish straight-line distances instead ([service areas](../local/service-areas.md)).

After a [domain migration](../technical/domain-migration.md), the profile's website link is a required follow-up. It is easy to leave pointing at the old host for months.

## The invisible-profile failure mode

**A business can be live, operating, and receiving reviews while its profile is effectively invisible to Google's own local search.** This is the finding that reframed our local engagement, and it is not visible from the dashboard.

What we observed (2026-07, headsupoutdoorservices.com): a Places **text search returned `ZERO_RESULTS` for the business** across every identity string — name alone, name + city, street address, phone number — with a 30 km location bias applied, while direct competitors in the same city resolved instantly on the same queries and the same key. The listing ID had to be recovered from the site's own `writereview?placeid=` link. The reading: an unverified, unpublished, or suspended profile — invisible in local search and Maps regardless of how good the website is.

### Diagnosis procedure

Run this in order; each step distinguishes a different cause.

1. **Human probe.** Search the exact business name in Google Maps, logged out (or in a private window). Then name + city. If a competitor-heavy list comes back without you, you have a profile problem, not a ranking problem.
2. **API probe** — one call per identity string, with a location bias around the real address:

    ```bash
    curl -s -X POST 'https://places.googleapis.com/v1/places:searchText' \
      -H "X-Goog-Api-Key: $PLACES_KEY" \
      -H 'X-Goog-FieldMask: places.id,places.displayName,places.formattedAddress' \
      -H 'Content-Type: application/json' \
      -d '{"textQuery":"<business name> <city>",
           "locationBias":{"circle":{"center":{"latitude":<lat>,"longitude":<lng>},
                                     "radius":30000.0}}}'
    ```

3. **Control query.** Run the identical call for a known competitor. If the competitor resolves and you don't, the API and key are fine — the listing is the problem.
4. **Recover the listing ID** from any `writereview?placeid=` or Maps CID link you already publish, then fetch it by ID. A listing that resolves by ID but not by text search is present but not *surfaced* — the classic unverified/suspended signature.
5. **Check the dashboard for a suspension notice** and the verification state. Suspensions are frequently caused by address/category edits, a virtual-office address, or category mismatch, and are appealed through the profile's own flow.
6. **Only then** look at ranking factors. Proximity, categories, and reviews decide *where* you rank among visible listings — they cannot make an unverified listing appear.

**How you know it's fixed:** the text-search probe returns your listing for name, name + city, address, and phone; the human Maps search shows you; and the profile's dashboard shows Verified with no notices.

## Gotchas

- **429 `RESOURCE_EXHAUSTED` on the Business Profile API is "not approved yet", not "slow down."** Retrying is wasted work. Apply for access early and use the browser lane in the meantime.
- **API keys never authenticate Business Profile or Search Console APIs.** OAuth or service-account only.
- **The Places API's 5-review cap is absolute** — no key tier, version, or pagination gets you more. Plan the review pipeline around it.
- **Verified in the dashboard ≠ findable in Maps.** Test findability with a real query; that gap is exactly the invisible-profile failure mode.
- **Claiming with a personal account** creates an ownership mess later. Use the business identity from the start; for client work, the client's account owns the asset.
- **Category inflation** — adding categories you don't serve to "cover more searches" dilutes relevance and can trigger review.
- **Site claims more cities than the profile does** — an easily-detected contradiction. Keep `areaServed` and declared service areas in sync.
- **Stale website link after a domain move** sends every profile click to a dead or redirecting host.
- **Recoloring or redrawing Google's "G"** on your site violates the identity guidelines; use it as published or not at all.
- **Waiting on GBP work until the site is done.** Verification lead time is the one thing you cannot compress — start it first.

## Related

- [Local Business](../local/index.md) — the full local stack this profile sits at the top of
- [Reviews — real ones only](../local/reviews.md) — the Dec-2025 placement policy and the live-sync architecture
- [LocalBusiness schema graph](../local/local-business-schema.md) — the sitewide node that `hasMap`/`sameAs`/NAP connect back to
- [Service-area pages](../local/service-areas.md) — the evidence standard for the cities you declare
- [Rich results](rich-results.md) — why ratings never go on the business node
- [Search Console](search-console.md) — the other Google property you verify, and the service-account patterns reused here
- [Off-site signals](../ai-search/offsite-signals.md) — the directory/NAP build-out that corroborates the profile
- [Playbook: launch a local business](../playbooks/local-business.md) — this chapter as an ordered sequence
- [Case study: Heads Up Outdoor Services](../case-studies/headsup.md) — where the invisible-profile finding came from

Source skills: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema) · [marketing-site-authenticity-audit](https://github.com/ever-just/agentskills/tree/main/skills/marketing-site-authenticity-audit) · [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization)
