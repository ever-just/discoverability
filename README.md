# Discoverability

The field guide to being found — by search engines, answer engines, and the agents that now pick software for people.

**Status:** Published · 61 chapters across 11 parts · public

[![docs](https://img.shields.io/github/actions/workflow/status/ever-just/discoverability/docs.yml?style=flat&color=1D1D1F&label=docs)](https://github.com/ever-just/discoverability/actions/workflows/docs.yml)

[Read the book](https://ever-just.github.io/discoverability/) ·
[Master checklist](https://ever-just.github.io/discoverability/start/master-checklist/) ·
[30-minute AI visibility audit](https://ever-just.github.io/discoverability/playbooks/ai-visibility-30min/) ·
[The spec that builds it](BLUEPRINT.md)

|  |  |
|---|---|
| **What it is** | A textbook on discovery across three audiences: crawlers, answer engines and AI agents |
| **Who it's for** | Founders, marketers and engineers shipping something that has to be found |
| **Live at** | [ever-just.github.io/discoverability](https://ever-just.github.io/discoverability/) |
| **Stack** | MkDocs Material · Python 3.12 · GitHub Pages, built `--strict` in CI |
| **Status** | Published · 61 chapters · ~106,000 words · sourced from shipped work |

## The problem it solves

Being findable used to mean one thing: rank on Google. It now means three things with three
different mechanics. Search engines rank pages. Answer engines — ChatGPT, Perplexity, AI
Overviews — return one answer and cite whoever earned it. AI agents skip the page entirely: they
look you up in an MCP registry, resolve your `/.well-known/` endpoints, and read your tool
descriptions the way a person reads a search snippet. Almost every guide online covers the first
surface and stops.

This book covers all three, and it is written the hard way round. Every chapter is distilled
from work that was actually shipped, measured or debugged on live sites — a SaaS launch, a set
of multi-tenant CMS sites behind reverse proxies, a local-services launch, a traffic-estimation
study. When something failed, the failure is written down. When a number is unmeasured, the
chapter says so.

## What it covers

- **Foundations** — how crawling, indexing and ranking work; how LLMs retrieve and cite;
  entities, E-E-A-T, and how to set a measurement baseline before you change anything.
- **Google** — Search Console, sitemaps and robots.txt, schema.org structured data, rich
  results, Business Profile, and keyword strategy including brand-name tokenisation.
- **Bing and beyond** — Bing Webmaster Tools, why Bing's index feeds ChatGPT, and IndexNow.
- **AI search (GEO/AEO)** — what content gets cited, AI-crawler allow-lists, the honest
  `llms.txt` reality check, and Ask-AI deep-link widgets.
- **AI agents** — the MCP Registry, the OAuth discovery chain, capability manifests and DNS-AID,
  tool descriptions that rank, and GitHub itself as a discovery surface.
- **Local business** — the LocalBusiness schema graph, FAQ schema from visible content, the
  real-reviews policy, service-area pages, and authenticity audits.
- **Technical** — DNS and domains, email trust (SPF/DKIM/DMARC), reverse-proxy CMS traps, domain
  migrations, and going invisible behind a WAF or bot challenge.
- **Playbooks and case studies** — end-to-end recipes for a SaaS launch, a local launch, a
  30-minute AI visibility audit and a quarterly operating cadence, plus the stories behind them.

## Quickstart

Read it at [ever-just.github.io/discoverability](https://ever-just.github.io/discoverability/),
or build the site locally:

```bash
uvx --with mkdocs-material mkdocs serve       # no install, serves on :8000
# or
pip install -r requirements.txt && mkdocs serve
mkdocs build --strict                          # what CI runs — broken links fail the build
```

## How it works

One MkDocs project. `mkdocs.yml` is the single source of navigation: every chapter appears in
`nav`, and `--strict` means an unlisted page or a broken internal link fails CI rather than
shipping. `docs/` mirrors the eleven parts one directory per part, each with an `index.md`
overview page.

```
docs/
  index.md            The three-audiences reframe — the entry point of the book
  start/              How discovery works, choose your path, master checklist, glossary
  foundations/        Crawl/index/rank, AI retrieval, entities and trust, measurement
  google/             Search Console, sitemaps, structured data, rich results, keywords
  bing/               Bing Webmaster Tools, IndexNow
  ai-search/          GEO/AEO — citation, AI crawlers, llms.txt, Ask-AI, visibility audits
  agents/             MCP Registry, OAuth discovery, manifests and DNS-AID, tool descriptions
  local/              LocalBusiness schema, FAQ schema, reviews, service areas, authenticity
  technical/          DNS, email trust, reverse-proxy traps, migrations, WAF invisibility
  playbooks/          SaaS launch, local launch, 30-minute audit, operating cadence
  case-studies/       The shipped work each framework came from
  appendix/           Tool directory, crawler registry, templates, skill index
mkdocs.yml            Theme, nav, markdown extensions — the build contract
BLUEPRINT.md          Sources, taxonomy, style contract, hygiene rules, rebuild prompt
requirements.txt      mkdocs >=1.6 · mkdocs-material >=9.5
```

## Development and deploy

New frameworks land first as reusable skills in
[ever-just/agentskills](https://github.com/ever-just/agentskills), then get distilled into
chapters here; the appendix's skill index maps each chapter back to its source skill.
[`BLUEPRINT.md`](BLUEPRINT.md) is the master spec — taxonomy, style contract, hygiene rules and
the exact prompt that rebuilds the book. Every push to `main` builds `--strict` and deploys to
GitHub Pages ([`.github/workflows/docs.yml`](.github/workflows/docs.yml)).

## License

No licence file yet. The prose is © 2026 EVERJUST; the code and configuration snippets inside
the chapters are free to copy and use. Open an issue if you want to republish a chapter.
