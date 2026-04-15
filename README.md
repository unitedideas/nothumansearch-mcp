# Not Human Search — MCP server

A remote MCP server for searching the agentic web. Discover APIs, services, and tools AI agents can actually call — ranked by agentic readiness score (0-100).

**Live endpoint:** `https://nothumansearch.ai/mcp` (streamable-http transport, no install)

**Registry listing:** [`ai.nothumansearch/search`](https://registry.modelcontextprotocol.io/v0/servers?search=nothumansearch)

## Install in one command

```bash
claude mcp add --transport http nothumansearch https://nothumansearch.ai/mcp
```

Any MCP-compatible client (Claude Code, Cline, Cursor, Zed, Goose, etc.) can connect to the same endpoint.

## Tools

| Tool | Purpose |
|------|---------|
| `search_agents` | Find agent-ready websites, APIs, and services by keyword, category, or minimum readiness score. |
| `get_site_details` | Get the full 7-signal readiness report for a specific domain. |
| `get_stats` | Index statistics: total sites, average score, top category. |

## What "agentic readiness" means

Every site in the index is scored 0-100 based on 7 signals that determine how cleanly an AI agent can interact with it:

- `llms.txt` (25 pts)
- `ai-plugin.json` (20 pts)
- OpenAPI spec (20 pts)
- Structured API (15 pts)
- MCP server (10 pts)
- `robots.txt` AI bot rules (5 pts)
- Schema.org markup (5 pts)

## Other interfaces

- **Website:** https://nothumansearch.ai
- **REST search API:** `GET https://nothumansearch.ai/api/v1/search?q=<query>`
- **On-demand check (CI-friendly):** `POST https://nothumansearch.ai/api/v1/check`
- **Embeddable badge:** `https://nothumansearch.ai/badge/{domain}.svg`
- **Submit a site:** `POST https://nothumansearch.ai/api/v1/submit`

## Manifest

```json
{
  "$schema": "https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json",
  "name": "ai.nothumansearch/search",
  "title": "Not Human Search",
  "description": "Search engine for AI agents. Find websites and APIs ranked by agentic readiness.",
  "version": "1.0.0",
  "websiteUrl": "https://nothumansearch.ai",
  "remotes": [
    { "type": "streamable-http", "url": "https://nothumansearch.ai/mcp" }
  ]
}
```

## License

The MCP server source lives in a private repo. This README is MIT-licensed — use the `claude mcp add` command freely.
