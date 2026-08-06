# AI Search (GEO/AEO)

**This part teaches you how to get cited by AI answer engines — ChatGPT, Perplexity, Google AI Overviews/AI Mode, Claude, Copilot — for the questions your buyers actually ask.** It is the book's centerpiece because it's where the most demand is moving and where the most snake oil is sold. Everything here is ranked by evidence: what measurably moves citations first, folklore last.

## GEO vs AEO vs SEO — the terms, straight

The industry uses three overlapping labels. They describe one shift:

| Term | Expands to | What it optimizes | The win condition |
|---|---|---|---|
| **SEO** | Search engine optimization | Ranking pages in Google/Bing results | Your URL in the top 10 |
| **AEO** | Answer engine optimization | Being *the answer* an engine extracts | Your sentence quoted in the answer box |
| **GEO** | Generative engine optimization | Being retrieved and **cited** by LLM-powered assistants | Your domain in ChatGPT/Perplexity citations |

In practice AEO and GEO are the same discipline (this book says "GEO"), and both sit **on top of** SEO, not beside it: answer engines retrieve from search indexes, so if you aren't crawlable and indexed you cannot be cited, period. See [How AI finds and cites](../foundations/ai-retrieval.md) for the mechanics.

## What actually moves citations

The evidence-based levers, in the order we prioritize them (each gets a chapter):

1. **Crawlability by AI bots** — one WAF toggle can erase you from every AI answer while your site looks fine to humans. We watched it happen: a live business at `site:` zero because a bot-challenge returned 429 to Googlebot, GPTBot, and PerplexityBot alike (measured, 2026-07). → [AI crawlers](ai-crawlers.md)
2. **Bing indexing** — ChatGPT search runs on Bing; industry studies put the overlap between ChatGPT citations and Bing's top organic results around **~87%** (external research, compiled 2026-07). Optimizing for Bing *is* optimizing for ChatGPT. → [Bing & Beyond](../bing/index.md)
3. **Answer-first, structured content** — comparison tables get extracted **~81% vs ~23%** for the same facts as prose; FAQ-structured pages earn roughly **3x** more citations; **~44%** of ChatGPT citations come from the first 30% of a page (external research, compiled 2026-07). → [Content that gets cited](content-that-gets-cited.md)
4. **Structured data** — rich schema appears on **~61%** of ChatGPT-cited pages vs ~25% of the baseline web (external research, compiled 2026-07). → [Structured data](../google/structured-data.md) for the how, [Content that gets cited](content-that-gets-cited.md) for the content-page types.
5. **Off-site presence where AIs retrieve** — **~85–93%** of AI citations point at third-party sources, not the brand's own site. Reddit alone is ~40%. → [Off-site signals](offsite-signals.md)
6. **Measurement** — a recurring query battery across the assistants, scored and trended. Everything above is a hypothesis until measured. → [Auditing your AI visibility](ai-visibility-audit.md)

## What's snake oil

Things sold as GEO that the evidence says to skip (details and receipts in [GEO fundamentals](geo-fundamentals.md)):

- **"Instant indexing" services and plugins.** Google's Indexing API is officially JobPosting/BroadcastEvent-only; off-label use risks revocation. The sitemap-ping endpoint has been dead since 2023.
- **llms.txt as a chat-citation lever.** ~97% of llms.txt files get zero bot requests (Ahrefs, ~137k domains) and Google says it doesn't use the file. It has one real audience — [coding agents reading docs sites](llms-txt.md).
- **"AI-specific" content rewriting, tiny-chunk fragmentation, secret "AI markup", paid mentions.** Google's own AI-search guidance names these as myths (documented).
- **Fake reviews, invented ratings, astroturfed forum posts.** Policy violations that also fail on mechanics — engines cite forums *because* they're authentic. The [honesty doctrine](../local/authenticity.md) is a ranking strategy here, not just ethics.

## How to read the evidence in this part

Every load-bearing claim in these chapters carries a tier:

- **Measured** — we shipped it on a live property and verified the result (the [case studies](../case-studies/index.md), with dates).
- **External research** — published studies (Ahrefs, the Princeton GEO paper, industry citation analyses), dated to when we compiled them. Percentages shift; treat them as direction and magnitude, not constants.
- **Documented** — vendor documentation or stated policy (Google, OpenAI, Anthropic, Perplexity).
- **Community-reported** — undocumented behavior (deep-link URL schemes especially) that we or others verified by testing. The most fragile tier; always date-stamped.

## The part map

| Chapter | What you get |
|---|---|
| [GEO fundamentals](geo-fundamentals.md) | The intent-vs-brand reframe, the three-surface model, the myth-killing ground truth, the prioritized lever list |
| [Content that gets cited](content-that-gets-cited.md) | Answer-first page structure, the pillar + glossary + how-to cluster model, content-page schema, freshness reality |
| [AI crawlers and crawlability](ai-crawlers.md) | The crawler roster and what each feeds, the robots.txt allow-list, verifying real access, blocking trade-offs |
| [llms.txt — the reality check](llms-txt.md) | What the data says, where it's actually real, the cheap-ship pattern, the decision rule |
| [The Ask-AI widget](ask-ai-widget.md) | Verified per-provider deep-link schemes, the self-contained-prompt rule, build notes |
| [Off-site signals](offsite-signals.md) | The sources answer engines actually cite and how to earn presence there without astroturfing |
| [Auditing your AI visibility](ai-visibility-audit.md) | The query battery, scoring rubric, gap-to-action mapping, re-audit cadence |

Start with [GEO fundamentals](geo-fundamentals.md) if you're new to this; jump straight to [the audit](ai-visibility-audit.md) if you want a baseline today.

## Related

- [How AI finds and cites](../foundations/ai-retrieval.md) — the retrieval mechanics under everything in this part
- [Bing & Beyond](../bing/index.md) — the index that gates ChatGPT
- [The 30-minute AI visibility audit](../playbooks/ai-visibility-30min.md) — the fast version of this part
- Source skills: [generative-engine-optimization](https://github.com/ever-just/agentskills/tree/main/skills/generative-engine-optimization), [ai-discoverability-audit](https://github.com/ever-just/agentskills/tree/main/skills/white-paper-writing/ai-marketing-skills/ai-discoverability-audit)
