# Auditing your AI visibility

**Everything else in this part is a hypothesis until you measure it.** The measurement is a fixed battery of ~30 queries run across five AI surfaces on a schedule, scored on four things per answer — are you mentioned, are you cited, is what it says accurate, and *who got cited instead* — then diffed against the previous round. This chapter is the full recurring audit: how to build the battery, how to score it, how to turn gaps into a ranked fix list, and the sheet format to keep it in. For a one-off 30-minute version including the crawlability probes, use [the 30-minute audit](../playbooks/ai-visibility-30min.md); this is what you run every quarter after that.

## Before you query: the pre-audit hypothesis

Ten minutes of reasoning first, written down, because it makes the results interpretable instead of just alarming.

1. **Entity clarity.** Is your brand name distinctive, or does it collide with a common phrase or another company? A name that tokenizes into generic words is a known failure mode — we found a branded query returning *zero* results for its own site because the engine split the dotted brand into two ordinary words (measured, 2026-07-15; [the tokenization crisis](../google/keyword-strategy.md)).
2. **Baseline expectation.** Given the company's age, size, and off-site footprint, do you expect strong, partial, or no recognition? A three-month-old domain that isn't mentioned is normal; a ten-year-old business that isn't is a finding.
3. **Competitive context.** Which competitors are likely well-represented in training data and in the sources engines cite? That's who you'll see instead of yourself.
4. **Positioning gap risk.** Write down the one sentence you *want* assistants to say about you. You'll diff the actual answers against it.

Record the hypothesis before running anything: *"Expect weak recognition. Main risk: category absence, not misattribution. Likely dominant competitor: X."* Being wrong about this is itself informative.

!!! warning "Check the gate before you blame the content"
    If a WAF, robots rule, or client-side rendering is blocking AI crawlers, every cell in this battery will read "not cited" and the diagnosis will be wrong. Run the [crawlability probe](../playbooks/ai-visibility-30min.md#minutes-06-fetch-the-site-the-way-bots-do) first. "Never mentioned" and "never fetchable" look identical from the answer side.

## Designing the query battery

The battery is fixed across rounds — that's what makes it a measurement rather than an anecdote. Aim for **25–30 queries in five classes**, weighted toward intent because branded queries are the easy half.

| Class | Count | What it tests | Pattern |
|---|---|---|---|
| **Brand** | 4–5 | Entity recognition and factual accuracy | "What is {brand}?", "What does {brand} do?", "Is {brand} any good?", "What do people say about {brand}?" |
| **Category** | 6–8 | Whether you appear in the consideration set at all | "Best {category} tools", "{category} for {segment}", "Recommend a {product} for {use case}" |
| **Problem intent** | 8–10 | The valuable half — asked by people who don't know you exist | The problem in the customer's own words, no brand: "how do I let my users use their own domain", "how much does snow removal cost per visit" |
| **Comparison** | 3–4 | Vendor-aware, bottom-of-funnel | "{incumbent} alternatives", "{you} vs {competitor}", "build vs buy {capability}" |
| **Local** (if applicable) | 4–5 | Map-pack-adjacent AI recommendations | "{service} near {city}", "who does {service} in {city}", "best {service} company in {city}" |
| **Expertise** (optional) | 2–3 | Authority and citation of your content | "Who are the experts in {niche}?", "What are best practices for {topic}?" |

Where the literal phrasings come from — don't invent them:

- **Search Console's query report.** Real strings people already reach you with, including the ones you rank #30–70 for. That striking-distance set doubles as battery input ([Measurement](../foundations/measurement.md), [Search Console](../google/search-console.md)).
- **Your ICP's own words.** Map each buyer segment to its job-to-be-done and write the literal query for each awareness stage — problem-aware, solution-aware, vendor-aware, and (for developer products) agent-aware: "MCP server for {thing}", "API to {do thing}".
- **Autocomplete, People Also Ask, and community threads.** Free, real, and phrased the way humans actually type.

Freeze the list, version it, and only add queries between rounds — never silently swap one out, or your trend line lies.

## The surfaces to run it on

| Surface | How to run it | Why it's in the set |
|---|---|---|
| **ChatGPT** (search enabled) | Temporary chat, logged out if possible | Largest assistant audience; retrieves via Bing |
| **Perplexity** | Fresh session | Different source pool entirely (~47% Reddit) |
| **Claude** | New chat, no project context | Training-corpus + live-search blend |
| **Gemini** | New chat | Google's assistant; sources differ from AI Overviews |
| **Google AI Overviews / AI Mode** | Incognito browser window | Appears above the classic SERP; feeds an enormous number of queries |

Yes, include Gemini even though it has no working deep-link scheme ([Ask-AI widget](ask-ai-widget.md)) — the widget and the audit are unrelated concerns.

**Session hygiene matters more than people expect.** Logged-in assistants with memory will happily mention your product because *you* keep asking about it. Use temporary/incognito sessions, don't personalize, and note which mode you used — the mode is part of the measurement.

## Scoring

Score two ways. The **raw counts** are the honest trend line; the **rubric** is the summary a stakeholder can read.

### Per-cell record (query × surface)

For each of the ~150 cells, record four things:

| Field | Values | Note |
|---|---|---|
| **Mentioned** | yes / no | Brand named anywhere in the answer |
| **Cited** | yes / no | Your domain appears as a linked source |
| **Accurate** | accurate / partial / wrong / n-a | Only meaningful when mentioned; "wrong" includes misattribution — wrong founder, wrong industry, confused with another company |
| **Sources cited** | domains | The whole list, not just yours. This is the most actionable field in the sheet. |

That fourth field is where the gap list comes from: the domains that get cited instead of you *are* your [off-site target list](offsite-signals.md).

### The rubric (six dimensions, 1–5)

| Dimension | 1 | 3 | 5 |
|---|---|---|---|
| **Recognition** | Assistants don't know the brand | Partial, vague knowledge | Accurate, detailed description |
| **Accuracy** | Wrong facts or misattribution | Mostly right, minor gaps | Fully accurate and current |
| **Sentiment** | Negative or skeptical | Neutral | Positive with specific reasons |
| **Category presence** | Never appears in category queries | Occasionally appears | Consistently in the top few |
| **Authority** | Never cited as a source | Occasionally cited | Regularly cited for expertise |
| **Competitive position** | Dominated by competitors | On par | Clearly leads in recommendations |

**Total /30** — 25–30 strong (maintain and expand) · 18–24 moderate (targeted fixes) · 10–17 weak (significant gaps) · under 10 invisible (foundational work). The band is a communication device; the raw counts are the truth.

### The competitive matrix

Run the **same** battery for your top two or three competitors — the same queries, same surfaces, same round. Different queries per company produces a comparison that means nothing.

| | You | Competitor A | Competitor B |
|---|---|---|---|
| Mentioned (n/30 per surface) | | | |
| Cited (n/30 per surface) | | | |
| Category presence 1–5 | | | |
| Authority 1–5 | | | |
| Sentiment 1–5 | | | |

## From gaps to fixes

Every failure pattern maps to a chapter. This table is the audit's actual output — a ranked work list, not a score.

| Symptom | Most likely cause | Fix |
|---|---|---|
| Nothing mentions you, anywhere, ever | Crawl gate: WAF challenge, `Disallow: /`, sitewide noindex, or client-side rendering | [AI crawlers](ai-crawlers.md), [Rendering and WAFs](../technical/rendering-and-waf.md) |
| ChatGPT blank, Perplexity and Google fine | Not in Bing's index | [Bing Webmaster Tools](../bing/bing-webmaster-tools.md), [IndexNow](../bing/indexnow.md) |
| Mentioned but never cited | Your pages aren't the retrievable evidence — no answer-first blocks, no tables, no schema | [Content that gets cited](content-that-gets-cited.md) |
| Brand query returns nothing / the wrong company | Entity problem: tokenizing brand name, duplicate or missing `Organization` node, thin `sameAs` | [Entities and trust](../foundations/entities-and-trust.md), [Keyword strategy](../google/keyword-strategy.md) |
| Facts about you are wrong or stale | Third-party sources carry old data; nothing authoritative to correct them | [Off-site signals](offsite-signals.md), [Structured data](../google/structured-data.md) |
| Absent from category queries; competitors present | No content targeting the category question; no third-party presence | [Content that gets cited](content-that-gets-cited.md), [Off-site signals](offsite-signals.md) |
| Local queries recommend others | Business Profile weak, reviews under the confidence bar, NAP inconsistent | [Google Business Profile](../google/business-profile.md), [Reviews](../local/reviews.md), [Off-site signals](offsite-signals.md) |
| Cited sources are Reddit threads and directories you're absent from | Off-site gap, exactly located | [Off-site signals](offsite-signals.md) |
| Agent-shaped queries ("MCP server for X") return nothing | Agent layer unclaimed | [AI Agents](../agents/index.md) |

Then triage by urgency, not by effort:

| Priority | Trigger | Timeline |
|---|---|---|
| **Critical** | Factual errors, misattribution, or total non-recognition; anything caused by a crawl gate | Now |
| **High** | Weak descriptions, absent from category recommendations, missing from a source class that dominates your queries | 30 days |
| **Opportunity** | Adjacent categories, expertise/authority queries, founder or team visibility | 90 days |

Never recommend keyword stuffing, fabricated reviews, misleading schema, or purchased mentions as a fix — they fail on policy *and* on mechanics ([what not to buy](geo-fundamentals.md#what-not-to-buy)).

## Cadence

| When | What you run | What you're asking |
|---|---|---|
| **Day 0 — baseline** | Full battery, all surfaces, plus the competitive matrix and a GSC + Bing Webmaster snapshot | What's true before we touch anything? |
| **Day 30** | Brand + category subset | Did the critical fixes (crawl gates, entity, factual errors) change recognition? |
| **Day 60** | Full battery, your brand only | Did the new content earn any citations yet? |
| **Day 90** | Full battery + competitive matrix | The real comparative re-audit; first defensible trend |
| **Quarterly thereafter** | Full battery + competitive matrix + re-verify the volatile facts | Where's the drift, and what broke? → [Operating cadence](../playbooks/operating-cadence.md) |

Set the dates before you deliver the baseline; an audit without a scheduled re-run is a document, not a measurement.

Two things belong in every quarterly round beyond the queries: **Bing Webmaster Tools' AI performance report**, which is the only first-party place to see Copilot/AI citations of your site (an audit of our own SaaS property graded observability an F precisely because this was never verified — measured, 2026-07-16); and a **re-verification pass** over the volatile facts in this part — crawler tokens, deep-link URL schemes, review policy, directory requirements.

## The tracking sheet

Two tabs. Keep them in a spreadsheet or a CSV in the repo — anywhere they'll survive and be diffable.

**Tab 1 — the query log** (one row per query × surface × round):

```csv
round_date,query_id,query_text,class,surface,mentioned,cited,accuracy,sentiment,sources_cited,notes
2026-08-06,Q01,"What is Example Product?",brand,chatgpt,yes,yes,accurate,positive,"example.com; g2.com",
2026-08-06,Q01,"What is Example Product?",brand,perplexity,yes,no,partial,neutral,"reddit.com; competitor.com","described as a DNS tool"
2026-08-06,Q12,"how do I let users use their own domain",intent,chatgpt,no,no,n-a,n-a,"competitor.com; stackoverflow.com",
```

**Tab 2 — the trend** (one column per round):

```markdown
| Dimension            | 2026-08-06 (baseline) | 30-day | 60-day | 90-day | Δ |
|----------------------|----------------------|--------|--------|--------|---|
| Mentioned (n/150)    | 23                   |        |        |        |   |
| Cited (n/150)        | 6                    |        |        |        |   |
| Accurate when mentioned | 17/23             |        |        |        |   |
| Recognition /5       | 2                    |        |        |        |   |
| Accuracy /5          | 3                    |        |        |        |   |
| Sentiment /5         | 3                    |        |        |        |   |
| Category presence /5 | 1                    |        |        |        |   |
| Authority /5         | 1                    |        |        |        |   |
| Competitive /5       | 2                    |        |        |        |   |
| **Total /30**        | **12**               |        |        |        |   |
```

Add a third artifact if you can: a **running list of the domains cited instead of you**, sorted by frequency. After two rounds it's the most valuable page in the whole audit.

## Self-critique before you deliver

Run this checklist against your own audit — it catches the failures we've made:

- [ ] Did I run at least **two** surfaces, and did I say which ones I couldn't run?
- [ ] Did I check for **misattribution** specifically, not just presence?
- [ ] Is the competitive comparison built from the **same** query set?
- [ ] Did I verify the crawl gate before concluding "content problem"?
- [ ] Are the recommendations specific and implementable, or generic "improve your SEO"?
- [ ] Are the re-audit dates real dates, with a stated thing to measure?
- [ ] If a prior audit exists, did I actually diff the scores?
- [ ] Did I re-verify each finding against the **live** site before writing it up?

State the gaps openly in the deliverable: *"Only Perplexity and ChatGPT were run; the Gemini and AI Mode columns are unmeasured."* Unmeasured is a legitimate result. Invented numbers are not.

## Gotchas

- **Answers vary run to run.** The same query on the same day can produce different answers and different citations. That's why you score ~150 cells and diff rounds, never react to a single answer. For load-bearing queries, run twice and note the disagreement.
- **Personalization flatters you.** Logged-in sessions with memory recall your product because you've discussed it. Temporary chats and incognito only.
- **Point-in-time audits rot.** In one of our own programs, **three of ten** critical findings had already been fixed by the time implementation started (measured, 2026-07). Re-verify every finding against the live site before acting on it — and date-stamp the audit prominently.
- **Confusing "not mentioned" with "not crawlable."** The single most common misdiagnosis. Probe the gate first.
- **Changing the battery between rounds.** Adding queries is fine (score them separately for one round); replacing them destroys the trend.
- **Auditing without a baseline.** Record GSC clicks/impressions/position and Bing Webmaster state *before* the program starts, or you'll never prove attribution ([Measurement](../foundations/measurement.md)).
- **Submission is not reception.** IndexNow 202s and sitemap submissions prove you asked, not that anything indexed or cited you. Only the battery and the Bing AI report tell you that.
- **Scoring sentiment from one answer.** Sentiment is the noisiest dimension; read all surfaces for a query before scoring it.

## Related

- [The 30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) — the fast, crawl-first version to run before this one
- [The operating cadence](../playbooks/operating-cadence.md) — where the quarterly re-run and re-verification live
- [Measurement and baselines](../foundations/measurement.md) — the search-side numbers that pair with this battery
- [GEO fundamentals](geo-fundamentals.md) — the levers this audit is testing
- [Off-site signals](offsite-signals.md) — where the "who got cited instead" list goes to work
- [Entities, E-E-A-T, and trust](../foundations/entities-and-trust.md) — the fix for wrong-facts and misattribution findings
- [Bing Webmaster Tools](../bing/bing-webmaster-tools.md) — the only first-party view of AI/Copilot citations
- Source skills: [ai-discoverability-audit](https://github.com/ever-just/agentskills/tree/main/skills/white-paper-writing/ai-marketing-skills/ai-discoverability-audit), [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization)
