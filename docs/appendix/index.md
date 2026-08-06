# Appendix

This is the book's reference layer: the pages you open *while doing the work*, not while learning it. Everything here is distilled from the chapters — the chapters carry the reasoning and the failure modes; the appendix carries the lookup tables and the copy-paste starting points. If a template or table here seems to contradict a chapter, the chapter wins (and please file an issue).

## What's in the appendix

| Page | What it is | Reach for it when… |
|---|---|---|
| [Tool directory](tools.md) | Every external tool the book references — Google's consoles and APIs, Bing, schema validators, DNS/email diagnostics, crawl probes, free traffic-estimation sources, MCP directories — each with what it's for, a URL, and a free/paid note | You know *what* to check but not *where* |
| [Skill index](skills-index.md) | The chapter → source-skill map into the public [ever-just/agentskills](https://github.com/ever-just/agentskills) repo | You want the operational, agent-runnable version of a chapter's method |
| [Templates](templates.md) | Eight copy-paste starters: LocalBusiness `@graph`, SaaS dual-node `@graph`, FAQPage, AI-crawler robots.txt, llms.txt, MCP `server.json`, policy-safe review markup, the Ask-AI deep-link row | You're shipping today and need a validated skeleton, not a tutorial |
| [AI crawler registry](crawler-registry.md) | The user-agent table: operator, what each bot feeds (training / search index / live fetch), robots.txt behavior, verification method, robots token — plus the curl-as-bot test pattern | You're writing robots rules, reading server logs, or debugging a bot wall |

## How to use it

1. **Templates are skeletons, not answers.** Every value is a placeholder (`example.com`, `YOUR-NAME`). Replace all of them, then validate — each template's usage note names the validator.
2. **Volatile facts are date-stamped.** Crawler behavior, deep-link URL schemes, and API scopes change; anything marked "as of 2026" should be re-verified before you build automation on it (the book re-verifies quarterly).
3. **Evidence tiers are explicit.** The crawler registry distinguishes operator-documented behavior from community-reported behavior. Never treat the second kind as a guarantee.
4. **The skills repo is upstream.** New frameworks land in [ever-just/agentskills](https://github.com/ever-just/agentskills) first, then get distilled into chapters here — the [skill index](skills-index.md) explains the loop.

## The fastest paths through

- **"Ship schema today"** → [Templates](templates.md) §1, §2, or §3 → validate per the note → [Structured data](../google/structured-data.md) for the reasoning.
- **"Are AI bots blocked?"** → [Crawler registry](crawler-registry.md) curl pattern → [Rendering, WAFs, and bot challenges](../technical/rendering-and-waf.md) if anything returns 403/429.
- **"Estimate a site's traffic for free"** → [Tool directory](tools.md) § Traffic estimation → [Measurement and baselines](../foundations/measurement.md) for the method.
- **"List our MCP server"** → [Templates](templates.md) §6 → [The MCP Registry](../agents/mcp-registry.md).

## Related

- [The master checklist](../start/master-checklist.md) — the cross-surface launch checklist these references support
- [Glossary](../start/glossary.md) — term definitions, one line each
- [Playbooks](../playbooks/index.md) — the ordered sequences that consume these templates
- [Measurement and baselines](../foundations/measurement.md) — how to know any of it worked
