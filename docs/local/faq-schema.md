# FAQ schema from visible content

Never hand-author `FAQPage` JSON-LD. **Extract the Q&A the page actually renders and generate the schema from it**, every time the page changes. Google's rule is that an FAQ answer must be present on the page for the markup to be valid, and the only way to guarantee that permanently is to make the visible content the single source and the schema a derived artifact. This chapter covers the extractor (both common FAQ markups), the node it produces, the embedding traps, and the verification loop that proves schema and page still agree.

## Why FAQPage is still worth shipping in 2026

Google's FAQ rich result is effectively gone for most sites (retired for the vast majority of results; the engagement research cited May 2026 as the point where it stopped being worth planning around — treat that as reported, not our own measurement). Ship it anyway, for a different audience:

| Consumer | What FAQPage does for you |
|---|---|
| Google Search | Little or no rich result any more. Structural clarity only. |
| AI answer engines | Question→answer pairs map 1:1 onto how people prompt an assistant. Industry research the case engagement relied on (2026-07, reported) put FAQ-schema pages at roughly 3x the citation rate and FAQ-structured content ~40% higher-weighted in ChatGPT source selection. |
| Bing | Ingests FAQPage — and Bing is the retrieval index behind ChatGPT search. |
| Your own QA | A deterministic FAQ pipeline makes schema-vs-content drift structurally impossible. That is worth the work on its own. |

So: keep the visible FAQ block (it is the thing that earns the citation), keep the markup (it is how machines parse it), and stop expecting a star-shaped SERP feature.

## Rule zero — the schema mirrors the page, not your intentions

The failure mode this chapter exists to prevent is **drift**: markup that was true when written and is a lie six weeks later. It is not hypothetical. On the case site, a quote tool's hand-written FAQ JSON-LD kept advertising prices after the visible prices were edited — the page said one thing and the structured data told Google those prices did not exist. The same class of bug hit review counts, which forked to three different values (47, 48, 51) across body copy, meta descriptions, and JSON-LD before a single-source pipeline landed ([reviews chapter](reviews.md)).

Hand-maintained FAQ JSON always drifts eventually, because:

- The content editor and the schema live in different places, and only one of them is visible to the person editing.
- Nobody re-validates markup after a copy tweak.
- Metadata fields (SEO title, meta description) are usually separate database fields that template edits never touch — so a sweep of the page body misses them.
- Multiple people (or agents, or sessions) edit the same page.

Determinism removes the human from the loop: change the page, regenerate, ship.

## The extractor

Real sites use at least two FAQ markups, often on the same site. Handle both, then merge in document order.

```python
import re, html, json

TAG = re.compile(r'<[^>]+>')

def clean(s: str) -> str:
    return html.unescape(TAG.sub('', s)).replace('\xa0', ' ').strip()

def extract_faq(page_html: str):
    """Return [(question, answer)] in document order, from the page's VISIBLE Q&A."""
    pairs = []

    # Pattern A — heading/paragraph:  <h3>Q</h3><p>A</p>
    for q, a in re.findall(r'<h3[^>]*>(.*?)</h3>\s*<p[^>]*>(.*?)</p>', page_html, re.S | re.I):
        pairs.append((clean(q), clean(a)))

    # Pattern B — accordion:  <button ...>Q</button> ... <div class="...answer...">A</div>
    for q, a in re.findall(
        r'<button[^>]*>(.*?)</button>\s*<div[^>]*>(.*?)</div>', page_html, re.S | re.I):
        pairs.append((clean(q), clean(a)))

    # de-dupe on the question, keep first occurrence, drop empties
    seen, out = set(), []
    for q, a in pairs:
        if q and a and q not in seen:
            seen.add(q)
            out.append((q, a))
    return out

def faq_node(pairs):
    return {
        "@type": "FAQPage",
        "mainEntity": [
            {"@type": "Question", "name": q,
             "acceptedAnswer": {"@type": "Answer", "text": a}}
            for q, a in pairs
        ],
    }
```

Adapt the two regexes to your actual markup — the point is not these exact patterns, it is that **the extractor reads the same HTML a crawler reads**. Run it against the fetched live URL, not a template file, so anything the CMS injects or strips is accounted for.

In the case engagement this extractor produced FAQ nodes for all 15 service pages in one pass (6 questions on most pages, 4 on the seasonal snow page — because that is what those pages actually showed).

## Where the node goes

Add `FAQPage` to the page's existing `@graph` alongside `Service` and `BreadcrumbList` (see [the schema graph chapter](local-business-schema.md)) — one JSON-LD block per page, not three:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Service", "@id": "https://www.example.com/services/lawn-care#service",
      "provider": { "@id": "https://www.example.com/#business" } },
    { "@type": "BreadcrumbList", "itemListElement": [] },
    { "@type": "FAQPage",
      "mainEntity": [
        { "@type": "Question",
          "name": "How often should a Minnesota lawn be mowed in July?",
          "acceptedAnswer": { "@type": "Answer",
            "text": "Weekly through the growing season. Cool-season grasses here grow fastest in June and September." } }
      ] }
  ]
}
```

Rules that keep it valid:

- **One `FAQPage` per page.** Two nodes describing the same page's FAQ is a conflict, not redundancy.
- **`FAQPage` is for Q&A the site author wrote.** If the page shows a user's question with community answers, that is `QAPage` with `suggestedAnswer`/`acceptedAnswer` — a different node type with different eligibility.
- **Answers go in as text, not as a summary.** `acceptedAnswer.text` may carry limited HTML, but it must be the answer the reader sees, not a shortened rewrite.
- **No answers that only exist in schema.** If you want a question in the markup, put it on the page first.

## Embedding without breaking the page

If your templates are XML (QWeb and family), serializing JSON into a `<script>` block is the step that silently corrupts pages:

```python
payload = json.dumps(graph, ensure_ascii=True, separators=(',', ':'))
payload = payload.replace('&', '&amp;').replace('<', '&lt;').replace('>', '&gt;')
block = f'<script type="application/ld+json">{payload}</script>'

# parse-validate the WHOLE template before pushing it
import xml.dom.minidom
xml.dom.minidom.parseString(full_template_xml)   # raises before you ship, not after
```

`ensure_ascii=True` keeps smart quotes and accented names from becoming encoding bugs. Escaping `&` is not optional — one unescaped ampersand inside a URL truncated a live page section in the case engagement, and a similar unescaped `&` split a form POST at the query string.

## The verification loop — how you know it worked

Do not trust the editor, the preview, or your own diff. Fetch the public URL and prove the two sides agree:

```bash
URL=https://www.example.com/services/lawn-care
curl -s "$URL?cb=$(date +%s)" -o /tmp/page.html

python3 - "$URL" <<'PY'
import sys, re, json, html
raw = open('/tmp/page.html').read()

# 1) what the page SHOWS  (reuse your extract_faq)
visible = [q for q, _ in extract_faq(raw)]

# 2) what the page CLAIMS
claimed = []
for m in re.findall(r'<script type="application/ld\+json">(.*?)</script>', raw, re.S):
    data = json.loads(html.unescape(m))
    for node in (data.get('@graph') or [data]):
        if node.get('@type') == 'FAQPage':
            claimed += [q['name'] for q in node.get('mainEntity', [])]

print('visible :', len(visible), 'claimed :', len(claimed))
print('MISSING FROM SCHEMA :', [q for q in visible if q not in claimed])
print('NOT ON PAGE         :', [q for q in claimed if q not in visible])
PY
```

You know it worked when both difference lists are empty and the counts match. Anything in `NOT ON PAGE` is a policy violation waiting to be found; anything in `MISSING FROM SCHEMA` is free extractability you are leaving on the table.

Then, once per page type:

1. **Schema Markup Validator** (validator.schema.org) — catches structural errors in every node, including ones Google no longer renders.
2. **Rich Results Test** — shows only what Google renders, so a quiet FAQPage result here is expected in 2026, not a failure.
3. **Re-run the diff after every content edit.** Wire it into whatever ships the page; a nightly job over the sitemap works too and catches edits made outside your pipeline.

## Gotchas

- **The block that renders is not always the block you edited.** CMSes with copy-on-write template forks can accept your write into a view that never renders — the case site had a "deployed" schema block invisible for weeks. Always verify against a fetched public URL ([Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)).
- **Meta fields are invisible to template greps.** SEO title and meta description usually live in separate database columns. A sweep that only reads view archs will miss stale claims sitting in the `<head>`. Grep the **rendered** page, head and body.
- **Client-side-only FAQs may not exist to a crawler.** If the Q&A is injected by JavaScript after a fetch, non-rendering crawlers (most AI fetchers) get an empty page and your extractor — if pointed at the raw HTML — will correctly produce nothing. Server-render the FAQ. See [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md).
- **Collapsed ≠ hidden.** Content inside an accordion is present in the delivered HTML and is fine to mark up. Content that only exists in the JSON-LD is not.
- **Do not paraphrase for the markup.** Shortening an answer "for schema" reintroduces exactly the drift you are eliminating, and it is now two claims to keep true instead of one.
- **Regenerate after price or policy edits specifically.** Those are the answers most likely to change and least likely to be re-validated — the case failure was a pricing FAQ.
- **Schema behind a bot challenge does not exist.** If a WAF returns 429 to crawlers, none of this is readable. Diagnose crawlability first.

## Related

- [LocalBusiness schema graph](local-business-schema.md) — the `@graph` this node joins
- [Reviews — real ones only](reviews.md) — the same anti-drift discipline applied to ratings
- [Service-area pages](service-areas.md) — city pages carry their own visible FAQ and their own node
- [Authenticity audits](authenticity.md) — schema that claims things the page cannot back
- [Structured data (schema.org)](../google/structured-data.md) — JSON-LD and `@graph` fundamentals
- [Rich results](../google/rich-results.md) — what Google actually renders today
- [Content that gets cited](../ai-search/content-that-gets-cited.md) — writing the questions worth marking up
- Source skill: [local-business-aeo-schema](https://github.com/ever-just/agentskills/tree/main/skills/local-business-aeo-schema)
