---
title: The field guide to being found
description: How to be found by Google, Bing, AI answer engines, and AI agents — distilled from real production work.
---

# Discoverability

**The field guide to being found — by Google, Bing, AI assistants, and AI agents.**

Being findable used to mean one thing: rank on Google. It now means three different things, with three different mechanics, and most advice online covers only the first:

<div class="grid cards" markdown>

- :material-magnify: **Search engines rank pages.**
  Googlebot and Bingbot crawl, index, and rank. You win with crawlability, structured data, content, and links. This is classic SEO — still the foundation everything else stands on.

- :material-message-question: **Answer engines cite sources.**
  ChatGPT, Perplexity, and Google's AI Overviews don't send you a ranked list — they give one answer and cite whoever earned it. You win by being the clearest, most retrievable source for *intent* questions, not brand searches.

- :material-robot: **AI agents call endpoints.**
  The newest audience isn't human at all. Agents look for your product in MCP registries, resolve your `/.well-known/*` endpoints, and read your tool descriptions like search snippets. If you're not registered and connectable, you don't exist to them.

</div>

## The one reframe that matters

**Branded discovery is easy. Intent discovery is the game.**

If someone searches your exact product name, you'll probably be found (though not always — see the [brand-name tokenization crisis](google/keyword-strategy.md)). The valuable, hard problem is being found by people — and AIs — who don't know you exist yet: *"best way to add custom domains to my SaaS"*, *"lawn care near Shakopee"*, *"tool that can connect a domain via API"*. Winning those queries requires deliberate work on all three surfaces at once. That work is what this book teaches.

## Where this book comes from

Every chapter here is distilled from real, shipped production work — not aggregated blog posts:

- **[customdomain.ai](case-studies/customdomain-ai.md)** — a SaaS product taken through a full AI-discoverability program: maximal schema.org graph, MCP registry presence, a GitHub discovery funnel, and a hard-won keyword strategy.
- **[Heads Up Outdoor Services](case-studies/headsup.md)** — a local service business taken through the local stack: LocalBusiness schema, live Google reviews, honest service-area pages, AI-crawler enablement.
- **[everjust.app tenants](case-studies/everjust-tenants.md)** — multi-tenant CMS sites behind reverse proxies, where a whole catalog of silent indexing traps was found and fixed.
- **The [ever-just/agentskills](https://github.com/ever-just/agentskills) repo** — where each framework was first captured as a reusable skill. The [skill index](appendix/skills-index.md) maps every chapter back to its sources.

When something failed, the failure is documented. When something is unmeasured, it says so. The book preaches authenticity as a ranking strategy, and it practices it.

## How to use it

- **Launching something?** Start with [Choose your path](start/choose-your-path.md) and the matching [playbook](playbooks/index.md).
- **Diagnosing a problem?** Jump to [Technical](technical/index.md) for indexing/infrastructure issues, or the [30-minute AI visibility audit](playbooks/ai-visibility-30min.md).
- **Learning the field?** Read [Foundations](foundations/index.md) first, then the surface-specific parts in order: [Google](google/index.md) → [Bing](bing/index.md) → [AI Search](ai-search/index.md) → [AI Agents](agents/index.md).

!!! tip "The master checklist"
    In a hurry? The [master checklist](start/master-checklist.md) is the whole book compressed into one actionable list, with links back to every chapter.
