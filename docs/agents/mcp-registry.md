# The MCP Registry

`registry.modelcontextprotocol.io` is the canonical upstream of MCP server listings: the community directories agents and users actually browse (mcp.so, Smithery, Glama, PulseMCP) mirror or ingest from it. That makes one correct publish there the single highest-leverage listing in the agent layer — you author one `server.json`, claim one namespace, and every downstream directory inherits your metadata. Do this before touching any individual directory.

## How the registry works

Three pieces, all yours to control:

1. **A `server.json`** — the machine-readable record describing your server: name, description, repository, and how to reach it (remote URL + transport, or package install instructions). The schema lives in [`github.com/modelcontextprotocol/registry`](https://github.com/modelcontextprotocol/registry).
2. **A namespace** — the part before the `/` in your server name, claimed one of two ways (DNS or GitHub, below). Namespacing is what stops squatters from listing a fake server under your brand.
3. **The publisher CLI** — `mcp-publisher`, which handles namespace authentication and pushes your `server.json` upstream.

As of 2026-07 the registry is young (it entered preview in September 2025) and most product categories have no listing at all. That is the opportunity.

## Step 1 — Author `server.json`

Skeleton for a hosted (remote) server, using customdomain.ai as the worked example:

```json
{
  "name": "com.customdomain/mcp",
  "description": "Buy, connect, and manage custom domains (DNS + SSL + edge) over MCP.",
  "repository": {
    "url": "https://github.com/CUSTOM-DOMAIN-APP/docs",
    "source": "github"
  },
  "version": "1.0.0",
  "remotes": [
    {
      "type": "streamable-http",
      "url": "https://app.customdomain.ai/mcp"
    }
  ]
}
```

Adapt the shape to your server: locally-installed servers declare `packages` (npm/PyPI/Docker install metadata) instead of `remotes`. Validate against the current schema in the registry repo before publishing — it evolves.

### Getting name and description right

This metadata set feeds *every* downstream directory, so treat these two fields like the title tag and meta description of the agent web:

- **Name** = `namespace/server-id`. Keep the server id short, generic, and stable (`mcp`, or a product noun). You will repeat this exact string across the manifest, directories, and docs — changing it later means re-claiming everywhere.
- **Description** = the search snippet agents and directory browsers rank you by. Write it for **intent**, not branding: lead with the verbs an agent's task would contain ("buy, connect, and manage custom domains"), name the category nouns (DNS, SSL), and say it's over MCP. No slogans, no "revolutionary", no exclamation marks — the same discipline as [tool descriptions](tool-descriptions.md).

A description that reads like a capability inventory outranks one that reads like a homepage hero, because the query it's matched against is a task.

## Step 2 — Claim your namespace

Two routes:

### Route A: reverse-DNS namespace (recommended for products)

Own `customdomain.ai` → claim `com.customdomain/*`. The namespace must be the reverse-DNS form of a domain you can prove you control, verified via a DNS TXT record:

```bash
# The publisher flow prints the exact TXT host + value — publish exactly that
# at your DNS provider. Example SHAPE (the value comes FROM the CLI; never invent it):
#   _mcp-registry.customdomain.ai.  TXT  "mcp-verify=<VALUE-FROM-CLI>"

mcp-publisher login dns --domain customdomain.ai
```

Wait for the TXT record to propagate (check with `dig TXT _mcp-registry.customdomain.ai +short`), then the login completes. See [Domains and DNS](../technical/domains-and-dns.md) if you're not sure who actually hosts your zone — publish the record at whoever `dig NS` says is authoritative, not whoever sold you the domain.

### Route B: GitHub namespace (`io.github.*`)

No domain verification required — authenticate with GitHub instead and claim `io.github.<username-or-org>/*`:

```bash
mcp-publisher login github
```

This is the right route for open-source servers without a product domain, and a fine fallback while your DNS access is blocked. For a commercial product, prefer Route A: `com.yourdomain/*` is self-evidently official in a way `io.github.someuser/*` is not, and it survives GitHub account changes.

## Step 3 — Publish

```bash
# From the directory containing server.json:
mcp-publisher publish
```

## Step 4 — Verify the listing took

Fetch your record back from the registry API — don't assume:

```bash
curl -s "https://registry.modelcontextprotocol.io/v0/servers?search=com.customdomain" \
  | python3 -m json.tool
```

You know it worked when your entry comes back with the right name, description, and remote URL. If the search is empty, the publish failed (most often: namespace verification never completed — recheck the TXT record).

## Step 5 — Mirror to the directories agents browse

The official registry is the upstream, but the browsing traffic is on the mirrors. After publishing upstream, claim or submit your server on each — reusing the **same name, same endpoint URL, same description** everywhere:

| Directory | Where | Submission |
|---|---|---|
| mcp.so | `mcp.so` | "Add server" flow |
| Smithery | `smithery.ai` | CLI: `smithery mcp publish https://app.customdomain.ai/mcp -n yourorg/your-server` |
| Glama | `glama.ai` | Add-server flow |
| PulseMCP | `pulsemcp.com` | Add-server flow |
| GitHub MCP Registry | GitHub | Listing flow in the GitHub ecosystem |
| awesome-mcp-servers | GitHub | PR to the list (community-curated — follow its contribution rules honestly) |

Each has its own review pace; the point is consistency. A directory card that shows a different endpoint than your registry entry is the kind of mismatch that produces listings that look published but won't connect — see the byte-identical rule in [Manifests and DNS-AID](manifests-and-dns.md).

## The gated first-party channels

Beyond the open registry there are two human-reviewed connector programs. Both are slower, and both are worth it if your users live in those clients:

- **Anthropic's connectors directory** (for Claude). As of 2026-07 the submission path runs through a Claude Team/Enterprise org's admin settings and the review expects: a stable public privacy-policy URL, per-tool `title` plus `readOnlyHint`/`destructiveHint` annotations, and a populated test account with reviewer instructions. The annotation requirement is one more reason to do [tool descriptions](tool-descriptions.md) properly before submitting.
- **OpenAI's ChatGPT apps submission portal.** Same idea, ChatGPT's directory.

Plan for review time on both — these are app-store-style processes, not instant listings.

## Gotchas

- **Submitting to directories while skipping the registry.** The mirrors ingest from the official registry; submitting downstream first means re-doing everything and fighting inconsistencies. Registry first, always.
- **Inventing the verification TXT value.** The publisher CLI prints the exact host and value. A hand-typed or guessed value fails verification and the publish is rejected. Copy exactly; retry after propagation.
- **Claiming a namespace you can't verify.** `com.customdomain/*` requires publishing the TXT record on `customdomain.ai` itself. You cannot claim a namespace for a domain you don't control, and you shouldn't reuse another product's namespace — the publish will be rejected.
- **Endpoint drift across surfaces.** The `remotes[].url` here, the `endpoint` in your capability manifest, and the `resource` in your OAuth metadata must agree byte-for-byte. A registry entry pointing at `/mcp` while the manifest says `/api/mcp` produces the hardest bug class: nothing errors, connections just fail.
- **Expecting answer-engine visibility from a registry listing.** Being in the MCP Registry makes you findable by *agents and MCP tooling*. It does nothing for ChatGPT Search or Perplexity citing your marketing pages — that's the [AI Search](../ai-search/index.md) part, with different levers entirely.
- **Treating the listing as done before a client connects.** A resolvable registry entry is necessary, not sufficient. The proof is a real MCP client completing the [OAuth chain](oauth-discovery.md) and listing your tools.

## Related

- [OAuth discovery chain](oauth-discovery.md) — the listing gets you found; this gets you connected
- [Manifests and DNS-AID](manifests-and-dns.md) — the consistency rule across all four surfaces
- [Tool descriptions that rank](tool-descriptions.md) — the description discipline this page's metadata shares
- [GitHub as a discovery surface](github-as-discovery.md) — the parallel funnel for the same audience
- [Domains and DNS](../technical/domains-and-dns.md) — publishing the verification TXT at the right provider
- [Launch a SaaS product](../playbooks/saas-launch.md) — where registry publishing slots into a launch
- Source skills: [mcp-server-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/mcp-server-discoverability), [agent-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/agent-discoverability)
