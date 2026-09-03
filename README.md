# Miriam Workspace Agent

Generated: 2026-09-01
App ID: `bcafde7e-0482-4b40-aa9a-9d3f0d3463c8`
Declarative agent schema: `v1.8`

## What is included

- `manifest.json`: Microsoft 365 app manifest
- `declarativeAgent.json`: read-only, evidence-first agent behavior
- `color.png`: 192 x 192 color icon
- `outline.png`: 32 x 32 transparent outline icon
- `TAVILY_SETUP.md`: safe MCP connection steps

## Important pre-provisioning edits

The developer URLs use the intentionally non-routable `example.invalid` domain. Replace them with real HTTPS website, privacy, and terms URLs before organizational publication. For development sideloading, Microsoft validation may still require reachable URLs.

## Tavily connection

This base package does not contain an API key and does not falsely claim that Tavily is already wired. In Microsoft 365 Agents Toolkit:

1. Open this project or create a declarative-agent project and copy these manifests.
2. Choose **Add Action** and then **Start with an MCP server**.
3. Enter `https://mcp.tavily.com/mcp`.
4. Prefer OAuth when the Toolkit and tenant complete the flow successfully.
5. Select only the required read-only research tools initially, preferably search and extract.
6. Review the generated `ai-plugin.json`, authentication settings, and updated manifest files.
7. Provision and start debugging, then verify one Tavily search call and its citations.

Do not put a Tavily API key into a URL committed to source control or into this ZIP.

## Validation performed

- JSON parsing: passed
- Required files present: passed
- Icon dimensions and transparency: passed
- ZIP root layout: passed
- SHA-256 inventory: see `SHA256SUMS.txt`

## Validation still required in your tenant

Run Microsoft 365 Agents Toolkit validation or `wiqd agent validate` after adding Tavily, because MCP discovery and authentication produce environment-specific generated files.
