# The Ask-AI widget

**An "Ask AI about us" row is a handful of `<a href>` tags that open the visitor's own assistant with a prompt about you already typed in — mechanically identical to a "Tweet this" button.** It's the one route to AI visibility you control outright (the user-supplied-context route from [How AI finds and cites](../foundations/ai-retrieval.md)), it takes an afternoon, and it fails in exactly one interesting way: if the prompt tells the assistant to *go read your site*, it will search instead of fetch, find nothing, and tell the user you don't exist. This chapter gives you the verified URL schemes, the prompt rule that fixes that, and the build details.

## The verified schemes

Each provider accepts the prompt in a query parameter, URL-encoded. As of **2026-07**, verified by direct testing (community-reported tier — see the caveat below):

| Provider | URL (`{Q}` = URL-encoded prompt) | Behavior | Confidence |
|---|---|---|---|
| **ChatGPT** | `https://chatgpt.com/?q={Q}` | Prefills and usually auto-submits on a direct click | High on the URL, medium-high on auto-run |
| **ChatGPT + web search** | `https://chatgpt.com/?hints=search&q={Q}` | Same, forcing the search tool | High |
| **Perplexity** | `https://www.perplexity.ai/search?q={Q}` | Auto-runs — it's a search engine | High |
| **Google AI Mode** | `https://www.google.com/search?udm=50&q={Q}` | Server-side AI answer on load, not a chat prefill | High |
| **Claude** | `https://claude.ai/new?q={Q}` | Prefills a new chat; auto-run unconfirmed | High on the URL, unknown on auto-run |
| **Grok** | `https://grok.com/?q={Q}` | Usually auto-submits; some flows need a manual send | Medium |

**Skip these** (shipping a dead-end icon is worse than shipping four working ones):

- **Gemini** — `gemini.google.com` ignores `?q=` and `?prompt=` natively. Google AI Mode above is the Google-shaped substitute.
- **Copilot** — its `?q=` parameter was removed/broken during 2026.
- **Meta AI, DeepSeek, Mistral** — no verified scheme. (`chat.mistral.ai/chat?q=` is plausible and unconfirmed; don't ship unverified.)

Two extras worth knowing: the legacy `chat.openai.com/?q=` 301s to `chatgpt.com` and the parameter survives the redirect; and a `&model=` hint is overridden back to the account's default model, so don't bother.

!!! warning "These schemes are undocumented — re-verify quarterly"
    None of these are vendor-documented APIs. They were found by testing (several surfaced originally in prompt-injection security research) and vendors change them without notice — Copilot's already broke once. Put a quarterly re-check in your [operating cadence](../playbooks/operating-cadence.md): click every icon, confirm the assistant opens with your prompt, and delete any that dead-end. A row of five icons where two 404 reads as neglect.

## The self-contained-prompt rule

This is the whole difference between a widget that works and one that embarrasses you.

**What we shipped first (2026-07-10, failed):**

> "Tell me about *Product* from *domain*. Please read `https://domain/llms.txt` first, then explain…"

**What actually happened on the user's first real click:** ChatGPT didn't fetch the URL — it ran a *search* (via Bing). The site was brand-new and unindexed, so the search returned nothing about the company. Worse, the literal token "llms.txt" in the prompt poisoned the query: the results were articles about the llms.txt *file format*. The assistant reported it couldn't find the product. Measured, first click, real user.

**The fix — bake the facts into the prompt:**

> "Explain *Product* (domain.com) in plain English. Grounding facts: it does X for Y; it works by Z; it's priced/licensed like so; the flagship flow is A → B → C. Tell me what it does, who it's for, and how the main flow works. More at https://domain.com"

The rule generalizes: **a deep link gives the assistant a prompt, not a page.** Assume it has zero prior knowledge of you and no reliable ability to fetch your URL. Everything the answer needs must be *in the prompt*, with your URL as a pointer for the curious, never as a dependency.

Prompt-writing specifics:

- **Two to four sentences of accurate grounding facts.** Enough for a good answer, short enough to survive URL encoding (ours ran 226 bytes → 308 encoded; well under any practical limit).
- **Only true facts.** This text will be read back to your prospect as if it were neutral third-party knowledge. Overclaiming here is a lie with extra steps — see [Authenticity audits](../local/authenticity.md).
- **Ask for the shape of answer you want** ("what it does, who it's for, how it works") rather than "is it good?", which invites the assistant to hedge.
- **Match your brand rules** — if your style bans em dashes or competitor names, the prompt is copy too.
- **One prompt sitewide, or one per page.** A footer row uses the product-level prompt; a docs page can carry a page-specific one ("Explain *concept* as implemented in *Product*. Grounding facts: …").

## Building it

It is a row of anchors. Resist frameworks — the best open-source implementation (`vladzima/copy2llm`, MIT) is a React component whose real value is its URL-scheme table; on a server-rendered site, hand-roll the fifteen lines.

```html
<div class="ask-ai">
  <span class="ask-ai__label">Ask AI about us</span>

  <a class="ask-ai__icon" target="_blank" rel="noopener noreferrer"
     aria-label="Ask ChatGPT about Example Product"
     href="https://chatgpt.com/?q=Explain%20Example%20Product%20(example.com)%20in%20plain%20English.%20Grounding%20facts%3A%20...">
    <svg viewBox="0 0 24 24" width="19" height="19" fill="currentColor" aria-hidden="true"><!-- brand mark path --></svg>
  </a>

  <a class="ask-ai__icon" target="_blank" rel="noopener noreferrer"
     aria-label="Ask Claude about Example Product"
     href="https://claude.ai/new?q=Explain%20Example%20Product%20...">…</a>

  <a class="ask-ai__icon" target="_blank" rel="noopener noreferrer"
     aria-label="Ask Perplexity about Example Product"
     href="https://www.perplexity.ai/search?q=Explain%20Example%20Product%20...">…</a>

  <!-- Google AI Mode: note the &amp; — a literal & breaks HTML/XML templates -->
  <a class="ask-ai__icon" target="_blank" rel="noopener noreferrer"
     aria-label="Ask Google AI Mode about Example Product"
     href="https://www.google.com/search?udm=50&amp;q=Explain%20Example%20Product%20...">…</a>
</div>
```

### Encoding, and the ampersand trap

Generate the encoded prompt once and paste it, or build it server-side:

```javascript
const prompt = "Explain Example Product (example.com) in plain English. Grounding facts: ...";
const q = encodeURIComponent(prompt);

const links = {
  chatgpt:    `https://chatgpt.com/?q=${q}`,
  claude:     `https://claude.ai/new?q=${q}`,
  perplexity: `https://www.perplexity.ai/search?q=${q}`,
  googleAI:   `https://www.google.com/search?udm=50&q=${q}`,   // raw & is fine in JS
  grok:       `https://grok.com/?q=${q}`,
};
```

`encodeURIComponent` handles the prompt body. The trap is one level up: **the `&` joining Google's two parameters must be written `&amp;` inside HTML or XML markup.** In a plain HTML file browsers forgive a raw `&`; in an XML-parsed template language (QWeb, XSLT, any CMS that validates its templates as XML) a raw `&` is a parse error or a silently truncated attribute. We've had an unescaped `&` in a different URL truncate a form field mid-value — same bug class, harder to spot. Validate the template with an XML parser before pushing.

### Icons

Use the official monochrome brand marks — Simple Icons (`cdn.jsdelivr.net/npm/simple-icons@latest/icons/{slug}.svg`), with `@lobehub/icons-static-svg` as the fallback. Simple Icons has periodically **removed AI-vendor marks** over trademark requests (OpenAI and Grok both 404'd on us mid-build, 2026-07), so vendor the SVGs into your repo rather than hotlinking a CDN path that can disappear. Render them inline with `fill="currentColor"` at ~19px so they inherit your footer's text color, each wrapped in an anchor with a real `aria-label`.

## Why this is legitimate

A reasonable objection: isn't stuffing a prompt into someone's assistant a form of prompt injection? No, and the distinction is worth stating in your own words when a security reviewer asks:

| Prompt injection | An Ask-AI link |
|---|---|
| Text hidden in a page that an *agent* reads and obeys without the user knowing | A visible button the *user* deliberately clicks |
| Attempts to override the assistant's instructions or the user's intent | Supplies a starting question the user can read, edit, or delete before sending |
| Targets the model's trust in retrieved content | Targets nothing — it's a URL with a query string, like a mailto link |

The user is the one asking. They see the prompt in their own chat box, on their own account, and can rewrite it. That's a share button, not an attack. (The reason the auto-submit behavior is *request-context gated* on ChatGPT — it checks how the navigation originated — is precisely that vendors hardened against the scripted, non-user-initiated version. A direct click passes; a hidden iframe shouldn't, and that's correct.)

## How you know it worked

1. **The markup rendered.** Fetch the live page and count the hrefs — don't trust the CMS preview: `curl -s https://example.com/ | grep -o 'chatgpt.com/?q=\|claude.ai/new?q=\|perplexity.ai/search?q=\|google.com/search?udm=50\|grok.com/?q=' | sort | uniq -c`. You should see one of each.
2. **Every icon opens something.** Click all of them in a fresh browser profile (logged-out for at least one pass). Each should land on the provider with your prompt visible in the input, and — for ChatGPT, Perplexity, Google, Grok — an answer already generating.
3. **The answer is right.** Read what each assistant says back. This is the real test: if the answer is vague or wrong, your grounding facts are too thin, not the link.
4. **No dead ends.** Any provider that opens to a blank chat with no prompt gets removed from the row until re-verified.
5. **Accessibility.** Tab to each icon; the `aria-label` should announce the provider *and* the subject.

## Gotchas

- **Prompts that depend on fetching.** The headline failure above. "Read our llms.txt first" and "check our website" both produce searches, and searches fail for unindexed sites. Bake the facts in ([llms.txt reality check](llms-txt.md)).
- **Shipping icons for Gemini or Copilot.** Both dead-end as of 2026-07. Users read a broken button as a broken product.
- **The CMS template cache.** On a compiled-template CMS the row can sit correctly in the database and still not render. We hit this exactly once: the markup was saved, the page served the old compiled template, and clearing caches through the app's own APIs didn't propagate across worker processes — a service restart did. The nuance we later confirmed: writes made through the CMS's own API/RPC layer hot-invalidate the template cache and render immediately; edits made by poking the database from a shell do not. Match the write path to the invalidation you need, and always verify against the **rendered** HTML. → [Reverse proxies and CMS traps](../technical/reverse-proxy-cms.md)
- **Editor sanitizers.** Some visual page editors strip or rewrite raw markup on save. Re-fetch the live page after any subsequent visual edit to confirm the row survived.
- **Assuming the widget fixes discoverability.** It only helps the visitors already on your site. Passive AI discovery still requires [crawlability](ai-crawlers.md) and [Bing indexing](../bing/index.md) — and until those land, the self-contained prompt is doing 100% of the work.
- **Letting the prompt drift from the product.** Prices, capabilities, and positioning change; the encoded string in your footer doesn't. Add it to the same review pass as your meta descriptions.
- **Over-placing it.** One row in the footer, plus optionally a docs-page variant, is enough. An Ask-AI button on every section reads as a gimmick and buries your real CTA.

## Related

- [How AI finds and cites](../foundations/ai-retrieval.md) — the user-supplied-context route this exploits
- [llms.txt — the reality check](llms-txt.md) — why "go read our llms.txt" is the wrong prompt
- [AI crawlers and crawlability](ai-crawlers.md) — what makes the *passive* half work
- [Content that gets cited](content-that-gets-cited.md) — the grounding facts, written once, reused everywhere
- [The operating cadence](../playbooks/operating-cadence.md) — where the quarterly scheme re-verification lives
- [customdomain.ai (SaaS)](../case-studies/customdomain-ai.md) — the build, the failed first click, and the fix in context
- Source skill: [llm-deeplink-widget](https://github.com/ever-just/agentskills/tree/main/skills/llm-deeplink-widget)
