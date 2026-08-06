# Tool descriptions that rank

Once an agent connects to your server, a second discovery problem starts: of all the tools available to it — yours plus every other connected server's — which one does it call for the task at hand? The model decides by reading tool names and descriptions, which makes them search snippets in a ranking you can win or lose. Write them for **agent intent** — the task phrasing a user would delegate ("connect a domain", "create an invoice") — not for marketing, and your tools get picked; write them vaguely and a technically perfect server sits idle.

## The mechanics: what the agent actually sees

When a client session opens, the agent receives your tool list: for each tool a `name`, a `description`, an input schema (with per-parameter descriptions), and optional annotations. At task time, the model matches the user's request against that text. There is no PageRank here — the "ranking signal" is almost entirely **semantic match between your description and the task phrasing**, plus whatever trust the annotations convey. That has two consequences:

1. Your descriptions compete with every other connected server's descriptions, every time.
2. You can do keyword research for this the same way you would for search: enumerate the task phrasings your users delegate, and make sure each phrasing appears in the tool that serves it.

## Write descriptions for intent

The formula that works, per tool:

> **What it does** (first sentence, active verb) · **when to use it** (the trigger phrasings) · **inputs/outputs** (what it needs, what comes back) · **side effects** (what changes in the world).

Compare:

=== "Weak (gets skipped)"

    ```json
    {
      "name": "domain_tool",
      "description": "Powerful domain management capabilities for modern applications."
    }
    ```

=== "Strong (gets picked)"

    ```json
    {
      "name": "domains_check_availability",
      "description": "Check if a domain name is available to register. Use when the user asks whether a domain is taken, wants to buy a domain, or is choosing between candidate names. Input: a fully-qualified domain name. Returns availability and, if available, purchase pricing. Read-only."
    }
    ```

The weak version contains zero task phrasings — no sentence a user would actually say maps onto it. The strong version contains three. This is exactly the answer-first discipline from [content that gets cited](../ai-search/content-that-gets-cited.md), applied to a machine reader.

Guidelines that follow from the mechanics:

- **Lead with the verb + object the task would contain.** "Create an invoice for a customer", "Connect a custom domain to a site". If your users say "hook up a domain", include that phrasing too.
- **State when to use it — and when not to.** "Use for X. For Y, use `other_tool` instead." Disambiguation text prevents the agent from picking your wrong tool, which reads as *your product failing*.
- **Name inputs and outputs concretely.** Models plan multi-step work; knowing a tool returns an `invoice_id` lets the model chain it into the next call.
- **Declare side effects plainly.** "Sends the email immediately", "Charges the stored payment method". Well-behaved agents (and connector reviewers) need this to gate confirmation.

## Naming conventions

Names are matched too, and they're what shows up in logs, configs, and the agent's own reasoning:

- **`noun_verb` or `verb_noun`, consistently — pick one scheme and hold it.** `domains_check_availability`, `domains_suggest`, `invoices_create`. Consistency lets the model infer the family pattern.
- **Prefix by resource, not by brand.** The server is already namespaced (clients expose tools as `mcp__yourserver__toolname`); wasting name characters on your product name buys nothing.
- **One job per tool.** A `manage_domains` tool with a `mode` parameter ranks for nothing — split it. Small, single-purpose tools with sharp descriptions beat kitchen-sink tools every time.
- **Split reads from writes.** Separate `invoices_get` from `invoices_create`. This isn't just hygiene — it's what lets you annotate honestly, next.

## Annotations: the trust layer

The MCP spec defines per-tool annotations (documented behavior; support varies by client as of 2026-07): `title` (human-readable display name), `readOnlyHint`, `destructiveHint`, `idempotentHint`, and `openWorldHint`. Two of them are effectively mandatory in practice: **Anthropic's connector-directory review expects every tool to carry `title` plus `readOnlyHint` or `destructiveHint`** — and well-behaved agents use them to decide what needs user confirmation.

```json
{
  "name": "domains_purchase",
  "title": "Purchase domain",
  "description": "Buy an available domain and attach it to the current project. Use only after domains_check_availability confirms availability and the user has confirmed the purchase. Charges the account's stored payment method.",
  "annotations": { "readOnlyHint": false, "destructiveHint": true }
}
```

Annotate honestly. A write tool marked read-only will eventually execute without the confirmation the user expected — a trust failure that gets servers uninstalled and directory listings rejected.

## The server-level `instructions` string

Servers can ship a top-level `instructions` string the client injects as cross-tool context. Use it for what no single tool description can carry: the resource model ("a project has one site and many domains"), ordering rules ("check availability before purchase"), and global conventions ("all times UTC"). Keep it terse — it's context the model carries on every request, so every sentence must earn its tokens.

## The same discipline, one layer down: your API spec

Agents don't only arrive via MCP — coding agents also consume the REST API behind it. The same snippet discipline applies to that surface (per the source skill, as of 2026-07):

- Publish a **stable, public OpenAPI 3.x spec** with descriptive `summary` fields, meaningful `operationId`s (they become function names in generated clients), and request/response **examples** — an agent writing integration code ranks and picks endpoints by these exactly as it picks MCP tools by descriptions.
- Optionally serve `/.well-known/api-catalog` (RFC 9727) pointing at your API descriptions, so the spec itself is discoverable from the domain.
- Keep the OpenAPI operation descriptions and the MCP tool descriptions telling the same story — an agent that reads both and finds contradictions trusts neither.

## Anti-patterns

| Anti-pattern | Why it fails |
|---|---|
| **Marketing fluff** ("powerful", "seamless", "next-generation") | Contains no task phrasing; matches nothing a user delegates |
| **Vague verbs** (`do_thing`, `process_data`, `handle_request`) | Unrankable and unpickable — the model can't tell what it's for |
| **Homepage-speak instead of mechanics** | The reader is a model choosing a function to call, not a buyer |
| **Overlapping tools with near-identical descriptions** | The model picks between them arbitrarily; behavior looks flaky |
| **Kitchen-sink tool + `mode` parameter** | Dilutes the description across jobs; ranks for none of them |
| **Undeclared side effects** | Agents mis-gate confirmations; reviewers reject; users lose trust |
| **Description drift after behavior changes** | The model plans against the old contract and the calls fail |

## How you know it worked

Descriptions are testable. Don't ship on vibes:

1. **Connect a real client** (Claude Code / Claude Desktop / Cursor) to the live server via the [OAuth chain](oauth-discovery.md).
2. **Run a task battery** — 10–20 natural-language tasks phrased the way real users delegate, including phrasings that *shouldn't* trigger your tools.
3. **Score three things per task:** did the agent pick the right tool? With correct arguments on the first try? Did it avoid your tools on the negative cases?
4. **Fix the description, not the prompt.** Every miss is a snippet problem: add the missing task phrasing, sharpen the disambiguation, tighten the parameter descriptions.
5. **Re-run after every tool change**, and keep the battery in the repo next to the server code — it's the agent-layer equivalent of the [query battery](../ai-search/ai-visibility-audit.md) for answer engines.

Second-order verification: submit to a [gated connector directory](mcp-registry.md). The human review — annotations present, descriptions accurate, test account works — is a free external audit of exactly this layer.

## Gotchas

- **The description is a contract, not copy.** Agents plan multi-step work against what you wrote. If the description promises a return field the tool stopped emitting, downstream calls fail in ways users blame on *your product*, not the description.
- **Too many tools is a ranking problem too.** Fifty tools means fifty snippets competing for the model's attention, mostly against each other. Consolidate to the jobs users actually delegate; cut internal-admin tools from the public server.
- **Client support for annotations varies** (as of 2026-07). Treat hints as signals for the clients that honor them and reviewers who require them — never as a security boundary. Enforcement belongs server-side.
- **Mirror the vocabulary of your category, not your codebase.** If your internal name is "zones" but users say "domains", the description must say domains. The model speaks user, not your schema.

## Related

- [The MCP Registry](mcp-registry.md) — the server-level name+description that gets you found before tools are ever listed
- [OAuth discovery chain](oauth-discovery.md) — the connection these descriptions are read through
- [Manifests and DNS-AID](manifests-and-dns.md) — the manifest previews your tool list pre-connect
- [Content that gets cited](../ai-search/content-that-gets-cited.md) — the same answer-first discipline for human-facing AI surfaces
- [Auditing your AI visibility](../ai-search/ai-visibility-audit.md) — the sibling test battery for answer engines
- Source skills: [mcp-server-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/mcp-server-discoverability), [agent-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/agent-discoverability)
