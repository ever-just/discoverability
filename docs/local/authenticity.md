# Authenticity audits

Audit your own marketing site for fabricated claims before someone else does. The lie is almost never in the copy — it's in the **image versus its caption**, the testimonial that traces to nobody, and the badge you don't actually hold. This chapter is the detection method and the remediation doctrine, both proven on a live local-business site where an automated build pipeline had quietly invented case studies.

Authenticity is not a nicety here. Once you mark reviews up as [structured data](reviews.md), fabrication becomes a **policy violation with manual-action risk**; once your site says "verified reviews", an unverifiable testimonial becomes **legal exposure** under the FTC's rules on endorsements. And answer engines increasingly cite what they can corroborate — so honesty has become a ranking asset rather than a constraint.

!!! warning "Why this chapter exists"
    On headsupoutdoorservices.com, testimonials that matched no real customer — some machine-generated in earlier automated build waves — sat live alongside photos captioned as specific jobs they didn't depict, and a "BBB Accredited" badge the bureau's own profile contradicted. None of it was malicious; all of it was generated. It was caught by a dedicated late-stage audit that should have been a pipeline gate. See the [case study](../case-studies/headsup.md).

## The fabrication pattern catalog

Five patterns account for nearly everything you'll find. Learn to spot them by *mechanism*, because the copy always reads fine.

| Pattern | What it looks like | How to catch it |
|---|---|---|
| **Stock/AI image as "exact job"** | A generic or generated image captioned "this exact Shakopee job" | Fetch the image and look at it. Is it a jobsite photo, or a stock/AI asset? |
| **Image ↔ caption mismatch** | Caption describes a paver patio; image shows a lawn | Fetch and compare content against the claim |
| **One photo, many "exact" jobs** | The *same* photo captioned as several different specific jobs | Hash every image; duplicate hashes across different captions is the tell |
| **Invented testimonial** | A "Google review" pull-quote attributed only to a city — no real author, no findable source | Cross-check quote + attribution against the actual review platform |
| **Over-claim** | "No stock photos" / "100% real" / an accreditation badge, contradicted by the assets or the issuer | Test the claim against your own assets and the issuer's public profile |

## The core mechanism: fetch and compare

Unlike third-party verification, the source of truth here is **the site's own assets versus its own captions**. You must fetch and *view* the media — reading the HTML is not enough, because the text is usually the honest-looking part.

```python
import hashlib, requests

# items = [(image_url, caption), ...] enumerated from the page HTML
seen = {}
for img_url, caption in items:
    data = requests.get(img_url, timeout=20).content
    h = hashlib.sha256(data).hexdigest()
    if h in seen:
        print(f"REUSED: same photo captioned as both\n  - {seen[h]}\n  - {caption}")
    seen[h] = caption
```

Hashing catches reuse mechanically. The mismatch and stock/AI checks need a human or a vision model to actually **look at each image** and answer three questions:

1. Does the image content match its caption?
2. Is it a stock or generated asset?
3. Is it claimed as a **specific** job, or presented as illustrative?

Only the combination "generated/stock **and** claimed as specific" is a violation. Generated illustrations are perfectly legitimate when labeled as such.

### The city-limits test for location claims

A photo may carry a city claim only if its **EXIF GPS survives a geocode against actual municipal boundaries** — not the postal address, which routinely disagrees with the city line.

In the real audit this caught two distinct failures: a photo whose mailing address said one city while its coordinates sat in a neighboring township, and a landmark photo whose *camera position* was across a city line from the landmark it depicted. Both captions were rewritten to what was true. The same test governs which [service-area pages](service-areas.md) are allowed to exist at all.

## Verifying testimonials

For each testimonial, establish two things: **a real author** and **a verbatim body** traceable to a published source.

- **Real and published** → keep it, quoted verbatim, attributed exactly as the reviewer published it (city only if *they* published a city).
- **Real but private** (a customer email, say) → keep only with documented permission, and never mark it up as a `Review` in schema.
- **No traceable author or source** → it is fabricated, whatever its origin. Remove it.

!!! danger "The self-referential trap"
    If your site claims its reviews are "verified", every unverifiable testimonial converts a quality problem into a **compliance problem**. Either make the claim true or drop the claim.

## Confidence tiers and the retraction trail

Tag every claim, and keep the audit trail so the remediation is reviewable rather than a silent rewrite:

| Tier | Meaning | Action |
|---|---|---|
| **Verified** | Image matches caption; testimonial has real author + published source | Keep |
| **Suspect** | Mismatch, stock/AI origin, or city-only attribution | Flag; resolve before publishing |
| **Fabricated** | Reused-as-different-job, invented quote, or an over-claim the assets contradict | Retract with a strike-through record of the original claim and why it failed |

Preserving *why* something was retracted is what stops the same fabrication re-entering on the next content pass.

## The honest-generic remediation doctrine

The instinct is to delete everything. That's wrong — over-retraction is its own dishonesty, and it strips real evidence you've earned. Remediate by **reframing to what's true**:

- **Case studies** → convert "this exact [city] job" into an honest generic example ("an example of a typical paver-patio project"). The content survives; the false specificity doesn't.
- **Ratings and reviews** → keep only the real ones (real author + verbatim body). Delete invented pull-quotes outright.
- **Gallery** → real photos only.
- **Generated illustrations** → allowed and useful, but **never** captioned as a specific real job. Label them as illustrative.
- **Over-claims** → either drop the claim or make it true. In the real case, a false "BBB Accredited" badge was replaced with the accurate claim — an A+ *rating*, which the business genuinely held. The honest version was still an asset.
- **Badges and accreditations** → verify against the **issuer's** public profile, not your own marketing archive.

## Run the audit

1. **Enumerate** every case study, testimonial, gallery image, and authenticity claim, each with its caption or attribution.
2. **Fetch and compare** every image against its caption; hash all images to surface reuse.
3. **Verify** every testimonial for real author + published source.
4. **Geocode** every location claim against municipal boundaries.
5. **Verify** every badge against the issuer.
6. **Tier and retract** using the table above, preserving the trail.
7. **Remediate** per the honest-generic doctrine.
8. **Re-check the claims still hold.** If you kept AI illustrations, a "no stock photos" claim must now be gone.

### How you know it worked

- Every remaining "exact job" claim has a fetched photo you have personally looked at.
- No image hash appears under two different captions.
- Every testimonial resolves to a live, findable published review.
- Every location claim passes the city-limits geocode.
- Every badge matches the issuer's public record.
- The site's own authenticity claims are true of the remediated site.

## Gotchas

1. **Auditing text only.** The most common real failure is image-versus-caption. If you didn't fetch and view the assets, you didn't audit.
2. **Missing reused photos.** Without hashing, one photo serving as three different "exact jobs" reads as three fine case studies.
3. **Deleting real reviews to be safe.** Retract the fabricated ones, keep the genuine ones. Blanket deletion destroys legitimate social proof.
4. **Banning AI illustrations outright.** They're fine. The violation is *claiming one is a specific real job*. Relabel before you remove.
5. **Leaving a now-false authenticity claim.** Remediation changes the site — re-test the claims against the *new* state.
6. **Treating it as a launch-day task.** Generated content drifts into fabrication continuously. Make this a **gate in the content pipeline**, not a rescue mission — the single clearest lesson from the real audit.
7. **Schema-ifying before auditing.** Never mark up reviews as structured data until this audit has confirmed they're real; you'd be encoding the fabrication in a machine-readable, policy-enforced format.

## Related

- [Reviews — real ones only](reviews.md) — the policy rules this audit protects; audit *before* you mark up
- [Service-area pages](service-areas.md) — the city-limits test applied to whole pages
- [LocalBusiness schema graph](local-business-schema.md) — what you're allowed to encode once claims are verified
- [Entities, E-E-A-T, and trust](../foundations/entities-and-trust.md) — why fabrication is a discoverability risk, not just an ethics one
- [Off-site signals](../ai-search/offsite-signals.md) — directory and badge claims live here too
- [Heads Up Outdoor Services](../case-studies/headsup.md) — the audit that produced this chapter
- Source skill: [marketing-site-authenticity-audit](https://github.com/ever-just/agentskills/tree/main/skills/marketing-site-authenticity-audit)
