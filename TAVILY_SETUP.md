# Tavily MCP setup

Endpoint: `https://mcp.tavily.com/mcp`

Use Microsoft 365 Agents Toolkit to add this endpoint as an MCP action. Prefer OAuth. If the tenant/toolkit requires another method, store the credential in its secret store or environment configuration. Never embed the Tavily API key in `manifest.json`, `declarativeAgent.json`, source control, chat transcripts, or the distributable ZIP.

Initial allowed tools:
- Tavily search
- Tavily extract

Defer crawl, map, research, and any future write-capable operation until behavior and quota use have been reviewed.
