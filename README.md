# Discoverability

**The field guide to being found — by Google, Bing, AI assistants, and AI agents.**

📖 **Read the book: [ever-just.github.io/discoverability](https://ever-just.github.io/discoverability/)**

This repo is a comprehensive, textbook-style knowledge base distilled from real production work: launching and optimizing [customdomain.ai](https://customdomain.ai), [headsupoutdoorservices.com](https://headsupoutdoorservices.com), everjust.app tenant sites, and the frameworks captured in the public [ever-just/agentskills](https://github.com/ever-just/agentskills) repo. Nothing here is theory-first — every chapter traces back to something that was actually shipped, measured, or debugged.

## What it covers

| Part | What you learn |
|---|---|
| **Start Here** | The 2026 discovery landscape: search crawlers, answer engines, and AI agents as three distinct audiences |
| **Foundations** | How crawling/indexing/ranking works, how LLMs retrieve and cite, entities and E-E-A-T, measurement |
| **Google** | Search Console (UI + API), sitemaps/robots, schema.org structured data, rich results, Business Profile, keyword strategy |
| **Bing & Beyond** | Bing Webmaster Tools (and why Bing feeds ChatGPT), IndexNow |
| **AI Search (GEO/AEO)** | Getting cited by ChatGPT/Perplexity/AI Overviews, AI crawler allow-lists, the llms.txt reality check, Ask-AI deep-link widgets |
| **AI Agents** | The MCP Registry, OAuth discovery chains, capability manifests, DNS-AID, GitHub as a discovery surface |
| **Local Business** | LocalBusiness schema graphs, FAQ schema, real-reviews policy, service-area pages, authenticity audits |
| **Technical** | DNS/domains, email trust (SPF/DKIM/DMARC), reverse-proxy CMS traps, domain migrations, WAF/bot-challenge invisibility |
| **Playbooks** | End-to-end recipes: SaaS launch, local business launch, 30-minute AI visibility audit, operating cadence |
| **Case Studies** | The real stories behind the frameworks |

## Build locally

```bash
uvx --with mkdocs-material mkdocs serve
```

or

```bash
pip install -r requirements.txt && mkdocs serve
```

Deploys automatically to GitHub Pages on every push to `main` (see `.github/workflows/docs.yml`).

## How this book is maintained

The full specification — sources, taxonomy, style contract, hygiene rules, and the exact prompt that (re)builds this book — lives in [BLUEPRINT.md](BLUEPRINT.md). New frameworks land first as skills in [ever-just/agentskills](https://github.com/ever-just/agentskills), then get distilled into chapters here. The [skill index](https://ever-just.github.io/discoverability/appendix/skills-index/) maps every chapter to its source skills.
