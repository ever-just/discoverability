# GitHub as a discovery surface

GitHub is where coding agents and the engineers directing them go to find working software — and its search only indexes three fields: repository **name**, **description**, and **topics**. README bodies don't count unless a searcher explicitly adds `in:readme`. That one fact turns three small metadata fields into the entire ranking surface, and because almost no product teams treat GitHub as a discovery channel, most category queries are winnable at zero stars. This chapter is the org-as-funnel architecture we built for customdomain.ai (the `CUSTOM-DOMAIN-APP` org, 2026-07), and the GitHub-SEO facts it was built on.

## Why GitHub reaches agents at all

Researched for the customdomain.ai build, as of 2026-07:

| Fact | Why it matters |
|---|---|
| Repo search indexes **only name / description / topics** | Three fields carry everything; README quality affects conversion, not discovery |
| **Bing has privileged GitHub crawl** — and Bing powers much of ChatGPT search | A well-described public repo is retrievable at ChatGPT answer time ([why Bing matters](../bing/index.md)) |
| **MIT/Apache-licensed public repos enter LLM training corpora** | Your explanations become part of what future models "just know" |
| **AGENTS.md is auto-read by 20+ coding agents**; in-repo `llms.txt` has ~no consumers | One convention file makes a repo agent-legible; the other is a no-op here |
| **DeepWiki and Context7 re-serve repo docs into agent sessions** | Getting your docs repo into these pipelines puts you inside other people's coding sessions |
| `/blob/` pages are crawlable by search engines; `/tree/` and `/raw/` are **not** | Link to files, not directory views, when you want a page to rank |
| Topics are **exact-match** and most category topics are unclaimed | A topic is a tiny keyword monopoly, free to claim |
| **About-line keyword density beats stars** for ranking (the papermark "open-source X alternative" pattern) | A 0-star repo with the right description outranks a dead 20-star one |

The competitive finding that motivated the whole build: every target query ("custom domains for saas" and its neighbors) was near-empty — the top result was an abandoned repo — and the incumbent commercial competitor had **zero** GitHub presence. Whitespace, claimable in a day.

## The funnel architecture

One org, several small public repos, each doing one discovery job (the product repo itself stays private — this funnel is metadata and docs, not source):

| Repo | Job | Discovery mechanics |
|---|---|---|
| `.github` | Org front door | `profile/README.md` renders on the org page: hero, what-it-is, repo map, agent connection instructions |
| `docs` | **Public source of truth for product docs** | Content-only copy of the docs (no git history, secret-scanned before publish); what DeepWiki/Context7/agents ingest |
| `connect-domain-for-website-builders`, `-email-platforms`, `-ai-agents`, `-agencies` | Audience-targeted landing repos | Name *is* the query; each carries a substantial README, a few `docs/` deep-dives, `AGENTS.md`, MIT license |
| `awesome-custom-domains` | Category index | No awesome-list existed for the category — creating it makes you its curator; honestly lists competitors and open-source alternatives, which is exactly what makes it credible and linkable |

Every repo gets the same treatment: a keyword-dense description, 12–19 topics, and a homepage link deep into the relevant page of the product site.

Two design rules worth stealing:

- **Audience-targeted repos beat one mega-repo.** `connect-domain-for-ai-agents` matches an intent query verbatim in the *name* field — the strongest field there is. Four audiences, four repos, four queries owned.
- **The awesome-list must be honest to work.** Listing only yourself kills it in community review and nobody links it. Listing the real landscape (open-source tools, protocols, competitors) makes it the neutral index — which you happen to maintain, with your product present and well-described.

## Writing the three fields

**Name.** Where possible, the name is the query: `awesome-custom-domains`, `connect-domain-for-agencies`. Hyphenated, lowercase, boring. For the product repo itself, the product noun.

**Description (the About line).** This is your title-tag. Front-load capability keywords and the category, and use the "X alternative" capture pattern where a searched-for incumbent exists — searchers and models both query by the incumbent's name. Same intent-first discipline as [tool descriptions](tool-descriptions.md): capabilities, not slogans.

**Topics.** Exact-match labels, up to 20 per repo. Claim the category topics (many have one or zero repos), the protocol/standard names, the audience terms, and the incumbent-adjacent terms. This is the cheapest keyword claim on the internet as of 2026-07.

## AGENTS.md: make every repo agent-legible

`AGENTS.md` at the repo root is the emerging convention coding agents actually read automatically — 20+ agents as of 2026-07, versus effectively zero consumers for an in-repo `llms.txt`. In a discovery-funnel repo, use it to tell a visiting agent what the product is, where the real docs live, how to connect (the MCP endpoint and auth route), and what *not* to assume. An agent that lands in your repo mid-task and finds an AGENTS.md keeps moving toward your product instead of bouncing.

The same file convention serves your product repos and examples; write it once per repo, keep it short, point it at canonical docs. ([What goes in llms.txt, and why it mostly doesn't matter, is covered here](../ai-search/llms-txt.md).)

## The docs repo: public source of truth

Publishing your product docs as a standalone public repo is the highest-value move in the funnel, because docs are what agents, DeepWiki, Context7, and training corpora actually consume. The pattern we shipped (2026-07, verified):

1. **Copy content only.** Export the docs pages (Markdown/MDX) into the public repo *without git history* — history is where leaked secrets and internal references live.
2. **Secret-scan before the first push**, then keep scanning in CI. Treat the repo as adversarially read (it will be).
3. **One-way sync, humans merge.** A workflow in the docs repo pushes content changes to a branch of the private product repo (never to its main); engineers review and merge. Auth via a scoped deploy key held as an Actions secret. The docs repo is canonical; the product repo consumes.
4. **Declare canonicality** in the product repo (a `DOCS.md` pointing at the public repo) so contributors don't fork the truth.

Then register the docs repo with the re-serving pipelines (Context7, DeepWiki) so your docs answer questions inside other people's agent sessions.

## The org profile README

The `.github` repo's `profile/README.md` is the org's front page — the one place a human or agent who found *any* repo gets the full map. Ours carries: what the product does in one line, a demo GIF, an agent-connection tip block (the MCP endpoint and how to attach it), a repo map table, and an FAQ. Shipped and verified rendering 2026-07.

GitHub's Markdown sanitizer will fight you; verified rendering facts from that build:

- `style`, `class`, `script`, `iframe`, `video`, and `marquee` are **stripped**. No custom CSS, no embeds.
- Video only works via GitHub's own user-attachments upload URLs — a committed `.mp4` won't render. A GIF committed to the repo works fine.
- Logo walls: composite them into a single image, or use an HTML table of images. Positioning hacks don't survive the sanitizer.
- Markdown inside `<details>` or `<td>` needs blank lines around it or it renders as literal text.

## Finding your whitespace queries

Before building repos, build the query matrix — the same discipline as [keyword research](../google/keyword-strategy.md), run against GitHub's own search. For each capability your product serves:

1. **Name the concept three ways:** the canonical term, the practitioner phrasing, and the topic-slug guess ("custom domains for SaaS" / "connect a domain" / `domain-connect`).
2. **Check the topic first** — `github.com/search?q=topic:<slug>&type=repositories` and the topic page itself. Topics are human-curated and low-noise; an unclaimed or one-repo topic is a free monopoly.
3. **Then phrase-search the strong fields:** `"custom domains for saas" in:name,description` — quoted phrases beat keyword soup.
4. **Score the competition honestly:** for each hit, record stars *and* last push. A dead 20-star repo at #1 means the query is winnable; an active 5,000-star repo means pick a different query.
5. **Record the matrix** (query → top hit → verdict) so you can re-run it later as your rank check.

Useful for scripted checks (unauthenticated works at low volume):

```bash
curl -s "https://api.github.com/search/repositories?q=topic:domain-connect&sort=updated&order=desc&per_page=5" \
  | python3 -c "import json,sys; [print(r['stargazers_count'], r['full_name'], '-', (r['description'] or '')[:80]) for r in json.load(sys.stdin)['items']]"
```

Mind the API's limits: ~10 requests/min unauthenticated, 1,000 results max per query, and `sort` goes in the URL parameter — not inside the query string.

## Verify the funnel

You know the build worked when, checked from a logged-out browser or curl (not your own session):

- [ ] Each target query returns your repo on page one of `github.com/search?type=repositories`
- [ ] Each claimed topic page lists your repo
- [ ] The org page renders the profile README fully — every image resolves, no literal-markdown blocks
- [ ] `AGENTS.md` fetches raw from each funnel repo (agents fetch it unauthenticated)
- [ ] The docs repo appears in Context7/DeepWiki lookups after registration
- [ ] Repo descriptions and homepage links are populated on every public repo — no blank About lines

Then set the re-check cadence: the query matrix and traffic snapshot monthly, the render check after any README change.

## Measuring honestly

- **Traffic:** each repo's Insights → Traffic shows views, unique visitors, clones, and referring sites over a rolling 14-day window (documented GitHub feature). Export it periodically if you want history — GitHub doesn't keep it.
- **Search position:** re-run your target queries (`github.com/search?q=...&type=repositories`) on a cadence and log rank, the same discipline as a [SERP baseline](../foundations/measurement.md).
- **Stars are marketing, not usage.** Fine as a proxy for reach of a launch, useless as proof of adoption — and when *evaluating* other repos, always pair stars with recency (`pushed:`), since dead popular repos abound.
- **Our own outcomes, stated per this book's rules:** the funnel above shipped and rendered correctly as of 2026-07; its long-term ranking and referral outcomes are **unmeasured as of 2026-08**. The verified claims are the mechanics (what GitHub indexes, what renders, what was empty), not a traffic result.

## Gotchas

- **Optimizing the README while the description is generic** — the single most common mistake, and it's backwards: search never reads the README. Fix name/description/topics first; the README's job is converting whoever arrives.
- **`in:readme` matches forks' inherited READMEs**, so query results overstate competition; check the fork badge before deciding a niche is taken.
- **`sort:stars` inside an API query string silently does nothing** — sorting is a separate `sort` parameter on the REST API and `gh` CLI.
- **Search caps at 1,000 results per query** and only sees default branches; narrow with qualifiers rather than paginating.
- **Don't buy or beg stars.** Fake momentum is detectable (star-history cliffs) and this book's [authenticity doctrine](../local/authenticity.md) applies to repos too: fabricated signals are a discoverability *risk*, not a boost.
- **Private things leak through public funnels.** Anything you publish — docs content, workflow files, images — gets read by scrapers and models. Copy content without history, scan for secrets, and assume every committed byte is permanent.

## Related

- [Off-site signals](../ai-search/offsite-signals.md) — GitHub as one of the surfaces answer engines retrieve and cite
- [The MCP Registry](mcp-registry.md) — the machine registry this funnel complements
- [Tool descriptions that rank](tool-descriptions.md) — the same three-fields discipline, inside the MCP session
- [Keyword and SERP strategy](../google/keyword-strategy.md) — picking winnable queries; the same whitespace logic applied to Google
- [Measurement and baselines](../foundations/measurement.md) — the baseline-before-changes discipline applied here to repo traffic
- Source skills: [mcp-server-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/mcp-server-discoverability), [agent-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/agent-discoverability), and the [ever-just/agentskills](https://github.com/ever-just/agentskills) repo generally
