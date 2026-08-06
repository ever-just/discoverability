# Manifests and DNS-AID

Two frontier surfaces complete the agent-discovery stack: a **capability manifest** at `/.well-known/agent/mcp.json` that lets agents introspect your server before connecting, and **DNS-AID** — a record set (SVCB + TXT + `_index._agents` + TLSA under DNSSEC) that lets an agent resolve your domain straight to its MCP endpoint with DNS-native trust, no directory in the loop. Ship these *after* the [registry listing](mcp-registry.md) and [OAuth chain](oauth-discovery.md); they extend coverage, they don't replace the load-bearing pair.

!!! warning "Honesty first: this layer is emerging"
    As of 2026-07: the `/.well-known/agent/mcp.json` path is a **convention**, not a ratified standard — useful because DNS-AID's `cap=` parameter and directory cards need a URL to point at, but don't expect every client to fetch it. DNS-AID is an **IETF draft** (`draft-mozleywilliams-dnsop-dnsaid`, a Linux Foundation / IETF effort) with a reference implementation and few production deployments. Publishing correct records today is a bet on being in the first cohort agents find when resolvers ship — a cheap bet, but call it what it is. Everything on this page is documented or shipped-as-described in the source skills; adoption claims beyond that would be guesses, so we don't make them.

## The capability manifest

### What it answers

"What protocol and transport does this server speak, what tools does it expose, and what auth does it want — *before* I open a session?" Without it, an agent's only introspection path is to connect and find out, and DNS-AID and directory listings have no capability document to reference.

### Serve it

Over HTTPS, unauthenticated, as JSON, at `/.well-known/agent/mcp.json` on your base domain:

```json
{
  "protocol": "mcp",
  "name": "com.customdomain/mcp",
  "endpoint": "https://app.customdomain.ai/mcp",
  "transport": "streamable-http",
  "auth": {
    "type": "oauth2",
    "protected_resource_metadata": "https://app.customdomain.ai/.well-known/oauth-protected-resource"
  },
  "tools": [
    { "name": "domains_check_availability", "description": "Check if a domain is available." },
    { "name": "domains_suggest", "description": "Suggest available domains for a query." }
  ],
  "version": "1.0.0"
}
```

(`transport` is `streamable-http`, or `sse` for the legacy transport — declare what the endpoint actually speaks.)

### The byte-identical rule

The manifest's `name`, `endpoint`, `transport`, and `auth.protected_resource_metadata` must agree **byte-for-byte** with what your registry `server.json` and your OAuth chain declare. A registry entry pointing at `/mcp` while the manifest says `/api/mcp`, or a manifest declaring `sse` while the endpoint speaks `streamable-http`, produces the worst bug class in this part: a listing that looks published everywhere and connects nowhere, with no error anywhere.

One more stability constraint: the manifest's **SHA-256 is published in DNS** (the `cap-sha256` below). Every edit to the manifest body changes the hash, which means a DNS update too. Keep the manifest boring and stable; put volatile detail elsewhere.

### Verify

```bash
curl -s https://customdomain.ai/.well-known/agent/mcp.json | python3 -m json.tool   # valid JSON, right values
curl -s https://customdomain.ai/.well-known/agent/mcp.json | shasum -a 256          # note the hash for DNS
```

## DNS-AID: resolving a domain to its agent

DNS-AID (DNS-based Agent Identification and Discovery) answers the question directories can't: "given only `customdomain.ai`, where is its agent endpoint, and can I trust it without trusting any third party?" Trust comes from the DNS itself — DNSSEC signs the records, DANE/TLSA pins the endpoint's certificate — so an agent that resolves by domain needs no registry at all.

The record set for an MCP agent advertised at `mcp.customdomain.ai`, targeting the real endpoint host:

**SVCB — the service binding.** Points the agent name at the endpoint host, declares the protocol, and links the capability manifest plus its hash:

```text
mcp.customdomain.ai.  SVCB  1 app.customdomain.ai. alpn="mcp" port=443 \
    cap="https://customdomain.ai/.well-known/agent/mcp.json" \
    cap-sha256="<sha256-of-the-manifest-body>"
```

**TXT — machine-readable capabilities alongside the SVCB:**

```text
mcp.customdomain.ai.  TXT  "capabilities=mcp,streamable-http" "version=1.0.0"
```

**Index — a zone-level enumeration of the agents you publish**, so an agent can discover *all* of them from the apex:

```text
_index._agents.customdomain.ai.  TXT  "agents=mcp" "v=1"
```

**TLSA (DANE) — the endpoint's certificate pinned into DNS**, so the agent verifies the cert against the zone rather than only the public CA system:

```text
_443._tcp.mcp.customdomain.ai.  TLSA  3 1 1 <sha256-of-endpoint-cert-SPKI>
```

**DNSSEC — the part that makes any of it trustworthy.** The zone must be signed *and* the DS record placed at the registrar. Without the DS delegation, validating resolvers never set the AD (Authenticated Data) flag, and the whole record set is just unauthenticated text.

### Publish with the reference tooling

Don't hand-roll SVCB parameters and TLSA hashes. The reference implementation ([`github.com/infobloxopen/dns-aid-core`](https://github.com/infobloxopen/dns-aid-core)) ships a Python SDK, a CLI, and backends for the major DNS providers:

```bash
pip install "dns-aid[cloudflare]"    # backend extras: [cloudflare] / [route53] / [infoblox] / ...

# Provider credentials come from the environment — never inline them:
export CLOUDFLARE_API_TOKEN="<your-token-from-a-secret-store>"

dns-aid publish \
  --domain customdomain.ai \
  --agent mcp \
  --target app.customdomain.ai \
  --alpn mcp --port 443 \
  --cap "https://customdomain.ai/.well-known/agent/mcp.json"
```

### Verify with `dns-aid verify`

```bash
dns-aid verify mcp.customdomain.ai
```

You know it worked when the report shows: **AD flag = true** (DNSSEC chain intact through the registrar's DS record), **TLSA match = true** (the live cert matches the pinned hash), the endpoint healthy, and a non-zero score. Then close the loop on the manifest linkage:

```bash
curl -s https://customdomain.ai/.well-known/agent/mcp.json | shasum -a 256
# compare against the cap-sha256 published in the SVCB record — they must match
```

A failing AD flag with "correct-looking" records almost always means the DS record was never placed at the registrar — the zone is signed but nothing delegates trust to it.

## The cert-rotation tie-in

The TLSA record pins the *current* endpoint certificate. The moment that cert rotates — and modern certs rotate every 60–90 days — a stale TLSA breaks DANE, and a strict agent starts rejecting a perfectly valid endpoint. This is the operational trap that keeps most operators off DANE entirely, because cert renewal and DNS are usually separate systems owned by separate teams.

If one system in your stack both issues/renews TLS certs *and* writes DNS (a custom-domain product does, by design), wire the TLSA rewrite **into the renewal pipeline** so the record updates in the same step as the cert. If you can't unify them, at minimum run `dns-aid verify` after every renewal window and alert on TLSA mismatch. Operators who keep cert and TLSA in lockstep have a genuine differentiator here, not a checkbox — most cannot do it cleanly.

## Gotchas

- **Records on an unsigned zone are theater.** SVCB/TXT/TLSA without DNSSEC (signed zone + DS at the registrar) verify as untrusted; `dns-aid verify` shows AD = false and scores it down, and strict agents reject the whole set. Sign first, then publish.
- **A manifest edit silently breaks DNS verification.** New manifest body → new SHA-256 → stale `cap-sha256` in the SVCB record → verification failure. Treat manifest changes as a two-step deploy: edit the file, republish the DNS parameter.
- **Stale TLSA after cert rotation.** See above — the failure rejects a *valid* endpoint, which makes it maddening to diagnose from the server side, where everything looks green.
- **Do not spend a minute on `/.well-known/ai-plugin.json`.** That's the deprecated OpenAI ChatGPT-plugins manifest; no current agent framework discovers MCP through it. Serving it wastes effort and can mislead a reviewer into thinking discovery is handled.
- **Drift between the four surfaces.** Registry `server.json`, manifest, OAuth `resource`, and the SVCB target must stay in agreement as the product evolves. Put a consistency check in CI if you can: fetch all four, diff the endpoint/transport values.
- **`llms.txt` is not part of this stack.** It's a weak, complementary docs-index signal for coding agents, not an agent-connectivity surface — depth in [llms.txt — the reality check](../ai-search/llms-txt.md). Ship it if cheap; never count it as "agent discoverability handled."

## Related

- [The MCP Registry](mcp-registry.md) — the directory-based discovery path these records bypass
- [OAuth discovery chain](oauth-discovery.md) — what the manifest's `auth` block points into
- [Tool descriptions that rank](tool-descriptions.md) — the tool list the manifest previews
- [Domains and DNS](../technical/domains-and-dns.md) — who actually controls your zone, DNSSEC mechanics
- [Email trust (SPF/DKIM/DMARC)](../technical/email-trust.md) — the older precedent for "trust published in DNS"
- Source skills: [agent-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/agent-discoverability), [mcp-server-discoverability](https://github.com/ever-just/agentskills/tree/main/skills/mcp-server-discoverability)
