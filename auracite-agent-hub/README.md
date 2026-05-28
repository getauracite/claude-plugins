# AuraCite Agent Hub — Claude Code plugin

Connect Claude Code to your [AuraCite](https://auracite.de) AI-visibility (GEO) data: see how
ChatGPT, Gemini, and Perplexity mention, rank, cite, and recommend your brand — without leaving your
editor. Bundles the AuraCite MCP connector plus an `ai-visibility` skill.

## Install

### 1. Add the marketplace and install the plugin

**Today (local / pilot)** — from a clone of the AuraCite repo, with Claude Code launched at the repo
root:

```text
/plugin marketplace add ./plugins/auracite-claude-marketplace
/plugin install auracite-agent-hub@auracite
```

**Hosted marketplace** — from the AuraCite marketplace repo:

```text
/plugin marketplace add getauracite/claude-plugins
/plugin install auracite-agent-hub@auracite
```

### 2. Sign in — no API key needed (OAuth)

The connector uses **OAuth**: the first time Claude Code talks to it, it opens your browser to
`auracite.de`, you sign in and approve once, and Claude Code stores a scoped, read-only token. Run
`/mcp` (or just ask a visibility question) to trigger it. If you aren't signed in yet, the browser
takes you to the AuraCite login and back automatically. The token is bound to your default
project and inherits AuraCite's rate limits, CostGuard, and revocation.

That's it — then ask, e.g. *"How is my brand doing in ChatGPT vs. Perplexity this month?"*

### Alternative: static API key

Prefer not to use OAuth (CI, headless, shared machine)? Swap the connector to an API key. Create a
read-only key (`mcp:read`) in the AuraCite app under **API Keys** (admin-provisioned during the
pilot), then set `.mcp.json` to:

```json
{
  "mcpServers": {
    "auracite": {
      "type": "http",
      "url": "https://auracite.de/mcp/rpc",
      "headers": { "X-API-Key": "${AURACITE_MCP_TOKEN}" }
    }
  }
}
```

and export the key (never commit it):

```bash
export AURACITE_MCP_TOKEN="gp_..."   # macOS/Linux
```

```powershell
$env:AURACITE_MCP_TOKEN = "gp_..."   # Windows PowerShell
```

## What you get

- **MCP connector `auracite`** → read-only GEO tools: brands, mentions, citations, share-of-voice,
  visibility score, competitors, trends, per-engine breakdown, brand comparison.
- **Skill `ai-visibility`** (invoked as `/auracite-agent-hub:ai-visibility`) → guides Claude to
  answer visibility questions from real AuraCite data.

`tenant_id` and `project_id` are injected server-side from the token/key — a holder can only read
their own tenant's data. Mutating and cost-bearing tools are hidden from read-only access.

## Transport note (for maintainers)

`.mcp.json` uses Streamable HTTP (`type: http`) against `https://auracite.de/mcp/rpc` (JSON-RPC 2.0).
The endpoint advertises OAuth via `WWW-Authenticate` + `/.well-known/oauth-protected-resource`
(authorization-code + PKCE, dynamic client registration). If Claude Code's Streamable-HTTP handshake
doesn't negotiate cleanly, the legacy SSE transport is also live
(`backend/internal/adapters/http/api/mcp_sse.go`, `mark3labs/mcp-go`):

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

## Roadmap

- **Now:** one-click OAuth (browser login, no key) for read-only GEO tools.
- **Next:** publish to the public marketplace repo for one-line discovery; submit the OAuth server to
  the [Anthropic directory](https://claude.ai/directory) and this plugin to the `claude-community`
  marketplace.
- **Later:** `mcp:write`/`mcp:spend` scopes via the same OAuth flow (spend stays approval- and
  CostGuard-gated).

## License

Proprietary — © AuraCite (`LicenseRef-Proprietary`).
