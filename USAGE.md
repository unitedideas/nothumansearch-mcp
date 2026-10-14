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
| `limit` | int | Optional. Default 10, maximum 20. |
| `has_api` | boolean | Optional. Require a structured API signal. |
| `has_mcp` | boolean | Optional. Require an MCP server signal. |
| `has_openapi` | boolean | Optional. Require an OpenAPI specification. |
| `has_llms_txt` | boolean | Optional. Require an llms.txt file. |

The MCP response includes readable text and `structuredContent` with `results`, `total`, `search_id`, and `detail_actions`. Each result includes its canonical provider `url`.

Keep the returned `search_id` when opening a result. The corresponding `detail_actions` entry supplies the arguments for `get_site_details`. A lookup without `search_id` still returns details, but does not record a selection from that search.

### `get_site_details`

Returns the readiness report for a bare `domain`. Pass the optional `search_id` from the search that returned that domain to record the selection. Selection does not imply action interest or contact a provider.

### `get_stats`

Index stats: `total_sites`, `avg_score`, `top_category`. These describe the index, not customer use.

## 3. Practical recipes

### "Find me an email API that an agent can actually call"

```
search_agents(query="email", category="communication", has_api=true, min_score=50, limit=10)
```

Returns matching indexed services. Inspect the returned documentation before choosing a provider; a readiness score does not verify integration success.

### "What's the readiness of Stripe?"

```
get_site_details(domain="stripe.com")
```

Returns the 7-signal report. Agents use this to decide whether to call the API directly or escalate to a human for credentials.

### "Find MCP servers for finance"

```
search_agents(category="finance", has_mcp=true, min_score=40, limit=10)
```

Requires both the finance category and an indexed MCP signal. The keyword "mcp" alone does not enforce that filter.

### Search, then inspect a returned result

```text
search_agents(query="browser automation", has_api=true, limit=10)
get_site_details(domain="<returned domain>", search_id="<returned search_id>")
```

The angle-bracket values are placeholders. Use a domain and receipt from the same response, or invoke its supplied `detail_actions` entry.

Use capability filters such as `has_api` rather than adding generic capability words to the query. Current keyword search can match only part of a multiword query. Check that the result fits the requested task.

Any optional `record_action_interest` step requires explicit current intent from the human or company principal. Never infer that intent from a search or selection. It does not contact a provider or prove a sale.

## 4. Scoring (how the 0-100 score is built)

| Signal | Weight | What it proves |
|---|---|---|
| `llms.txt` | 25 | Site authors wrote agent-oriented docs |
| `ai-plugin.json` | 20 | Legacy but still useful agent manifest |
| OpenAPI spec | 20 | Deterministic API surface, agents can generate clients |
| Structured API | 15 | Evidence of REST/GraphQL endpoint |
| MCP server | 10 | Evidence of an MCP endpoint; client setup and provider access may still be required |
| `robots.txt` AI rules | 5 | Site is deliberate about agent access |
| Schema.org markup | 5 | Structured data extractable from any page |

Search results require a structured API, OpenAPI, ai-plugin, or MCP signal. An llms.txt file alone does not qualify a site for discovery results.

## 5. Other surfaces for humans

- **Web UI:** https://nothumansearch.ai
- **REST search:** `GET https://nothumansearch.ai/api/v1/search?q=<query>`
- **On-demand check:** `POST https://nothumansearch.ai/api/v1/check` (body: `{"url": "..."}`). Rate-limited to 10/hr per IP on the free tier.
- **Embeddable badge:** `https://nothumansearch.ai/badge/<domain>.svg`
- **Submit a site:** `POST https://nothumansearch.ai/api/v1/submit`

## 6. Design principles

1. **Zero-friction for agents.** No auth, no API key, no install. `claude mcp add` and you're querying.
2. **Agent-first index.** Signals, not popularity. Search requires an interaction signal; an llms.txt file alone is insufficient.
3. **Transparent scoring.** Every site shows why it got the score it got. No opaque ranking.
4. **Open protocol.** MCP is the primary interface. REST and the web UI are for humans and legacy agents.

## 7. Feedback & questions

Open an issue at https://github.com/unitedideas/nothumansearch-mcp or email `hello@nothumansearch.ai`.
