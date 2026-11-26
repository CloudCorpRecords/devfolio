# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted portfolio MCP server at krishparekhdev.com. All six captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `get_profile`, `get_hire_info`, `list_projects`, `list_posts`, `get_post`, `get_page` — read-only access to a published developer portfolio: profile, hire information, project list, article list and full article text, and top-level site pages, all returned as markdown.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `devfolio error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `KRISHP_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
