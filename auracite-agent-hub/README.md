# AuraCite Agent Hub — Claude Code plugin

Connect Claude Code to your [AuraCite](https://auracite.de) AI-visibility (GEO) data: see how
ChatGPT, Gemini, and Perplexity mention, rank, cite, and recommend your brand — without leaving your
editor. Bundles the AuraCite MCP connector plus an `ai-visibility` skill.

## Install

You need two things: the plugin (this repo) and a read-only AuraCite API key.

### 1. Add the marketplace and install the plugin

**Today (local / pilot)** — from a clone of the AuraCite repo, with Claude Code launched at the repo
root:

```text
/plugin marketplace add ./plugins/auracite-claude-marketplace
/plugin install auracite-agent-hub@auracite
```

**Public marketplace (planned)** — once these plugins are published to the public marketplace repo
`auracite/claude-plugins`, the one-line install becomes:

```text
/plugin marketplace add auracite/claude-plugins
/plugin install auracite-agent-hub@auracite
```

> The AuraCite monorepo is private, so `/plugin marketplace add <owner>/auracite` will not work for
> customers. Customer distribution goes through the dedicated **public** marketplace repo above. See
> [`../README.md`](../README.md) for the publish steps.

### 2. Get a read-only API key

In the AuraCite app, open **API Keys** and create a key with the **`mcp:read`** scope. Key creation
is currently limited to workspace admins — during the pilot AuraCite provisions an `mcp:read` key for
you on request. Copy it once (it starts with `gp_`); it is shown only at creation time.

### 3. Provide the key to Claude Code

Put the key in your environment as `AURACITE_MCP_TOKEN` (never commit it), then restart Claude Code
(or run `/reload-plugins`):

```bash
# macOS/Linux
export AURACITE_MCP_TOKEN="gp_..."
```

```powershell
# Windows PowerShell
$env:AURACITE_MCP_TOKEN = "gp_..."
```

Verify with `/mcp` — the `auracite` server should be listed with its tools. Then just ask, e.g.
*"How is my brand doing in ChatGPT vs. Perplexity this month?"*

## What you get

- **MCP connector `auracite`** → read-only GEO tools: brands, mentions, citations, share-of-voice,
  visibility score, competitors, trends, per-engine breakdown, brand comparison.
- **Skill `ai-visibility`** (invoked as `/auracite-agent-hub:ai-visibility`) → guides Claude to
  answer visibility questions from real AuraCite data.

`tenant_id` and `project_id` are injected server-side from the API key — a key holder can only read
their own tenant's data. Mutating and cost-bearing tools are hidden from `mcp:read` keys.

## Transport note (for maintainers)

`.mcp.json` uses Streamable HTTP (`type: http`) against `https://auracite.de/mcp/rpc` (JSON-RPC 2.0)
with `X-API-Key` auth. If Claude Code's Streamable-HTTP handshake doesn't negotiate cleanly with the
single-shot JSON-RPC endpoint, switch the plugin's `.mcp.json` to the legacy SSE transport, which is
a full MCP server (`backend/internal/adapters/http/api/mcp_sse.go`, `mark3labs/mcp-go`) and accepts
`Authorization: Bearer`:

```json
{
  "mcpServers": {
    "auracite": {
      "type": "sse",
      "url": "https://auracite.de/mcp/sse",
      "headers": { "Authorization": "Bearer ${AURACITE_MCP_TOKEN}" }
    }
  }
}
```

Confirm the transport against a live Claude Code session before public listing.

## Roadmap

- **Now:** read-only via API key (this plugin), local-path install for the pilot.
- **Next:** publish `auracite/claude-plugins` (public marketplace) for one-line install; self-service
  `mcp:read` key creation for customers (today key creation is admin-only).
- **Then:** OAuth 2.0 on the MCP server → browser login instead of pasting a key; submit the OAuth
  server to the [Anthropic directory](https://claude.ai/directory) and this plugin to the
  `claude-community` marketplace for one-step discovery.

## License

Proprietary — © AuraCite.
