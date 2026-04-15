# Using Not Human Search from your agent

This is a practical guide to wiring Not Human Search (NHS) into a Claude, Cursor, or Cline agent so it can discover agent-ready websites at runtime.

## 1. Install (one line)

```bash
claude mcp add --transport http nothumansearch https://nothumansearch.ai/mcp
```

Or any MCP-compatible client — the endpoint is `https://nothumansearch.ai/mcp` with `streamable-http` transport. No install, no API key.

## 2. The tools

### `search_agents`

Find agent-ready sites by keyword, category, or minimum readiness score.

| Argument | Type | Notes |
|---|---|---|
| `query` | string | Optional full-text query. Searches name, description, domain, tags. |
| `category` | string | Optional. One of: `ai-tools`, `communication`, `data`, `developer`, `ecommerce`, `education`, `finance`, `health`, `jobs`, `news`, `productivity`, `security`. |
| `min_score` | int | Optional. 0-100. Default 0. |
| `per_page` | int | Optional. Max results. Default 20. |

Returns a list of sites with `domain`, `name`, `description`, `agentic_score`, `category`, `tags`, and `url` (report page).

### `get_site_details`

Full readiness report for a single domain — returns the 7-signal breakdown (llms.txt, ai-plugin.json, OpenAPI, structured API, MCP server, robots.txt AI rules, Schema.org) plus tags and category.

### `get_stats`

Index stats: `total_sites`, `avg_score`, `top_category`. Useful for agents that want to brag about the size of the index they're searching.

## 3. Practical recipes

### "Find me an email API that an agent can actually call"

```
search_agents(query="email", category="communication", min_score=50)
```

Returns Resend, Postmark, SendGrid, Loops, etc. — ranked by how cleanly an agent can integrate.

### "What's the readiness of Stripe?"

```
get_site_details(domain="stripe.com")
```

Returns the 7-signal report. Agents use this to decide whether to call the API directly or escalate to a human for credentials.

### "Find all MCP servers for finance"

```
search_agents(query="mcp", category="finance", min_score=40)
```

Filters down to sites that expose an MCP server AND are finance-adjacent.

## 4. Scoring (how the 0-100 score is built)

| Signal | Weight | What it proves |
|---|---|---|
| `llms.txt` | 25 | Site authors wrote agent-oriented docs |
| `ai-plugin.json` | 20 | Legacy but still useful agent manifest |
| OpenAPI spec | 20 | Deterministic API surface, agents can generate clients |
| Structured API | 15 | Evidence of REST/GraphQL endpoint |
| MCP server | 10 | Native agent protocol — zero integration work |
| `robots.txt` AI rules | 5 | Site is deliberate about agent access |
| Schema.org markup | 5 | Structured data extractable from any page |

Sites without at least one strong signal (llms.txt, ai-plugin.json, OpenAPI, structured API, or MCP server) are filtered out of the index entirely — the NHS index is agent-first by construction.

## 5. Other surfaces for humans

- **Web UI:** https://nothumansearch.ai
- **REST search:** `GET https://nothumansearch.ai/api/v1/search?q=<query>`
- **On-demand check:** `POST https://nothumansearch.ai/api/v1/check` (body: `{"url": "..."}`). Rate-limited to 10/hr per IP on the free tier.
- **Embeddable badge:** `https://nothumansearch.ai/badge/<domain>.svg`
- **Submit a site:** `POST https://nothumansearch.ai/api/v1/submit`

## 6. Design principles

1. **Zero-friction for agents.** No auth, no API key, no install. `claude mcp add` and you're querying.
2. **Agent-first index.** Signals, not popularity. A tiny site with a perfect `llms.txt` ranks higher than a Fortune-500 site with zero agent signals.
3. **Transparent scoring.** Every site shows why it got the score it got. No opaque ranking.
4. **Open protocol.** MCP is the primary interface. REST and the web UI are for humans and legacy agents.

## 7. Feedback & questions

Open an issue at https://github.com/unitedideas/nothumansearch-mcp or email `hello@nothumansearch.ai`.
