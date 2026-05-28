# AuraCite Claude Code marketplace

This directory is a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces):
its `.claude-plugin/marketplace.json` catalogs the AuraCite plugins. The marketplace **root** is this
directory (the one containing `.claude-plugin/`), so plugin `source` paths resolve relative to it.

Plugins:

- [`auracite-agent-hub`](./auracite-agent-hub) — MCP connector + `ai-visibility` skill for AuraCite
  GEO data. See its [README](./auracite-agent-hub/README.md) for install + API-key setup.

## Install locally (today)

From a clone of the AuraCite repo, with Claude Code launched at the repo root:

```text
/plugin marketplace add ./plugins/auracite-claude-marketplace
/plugin install auracite-agent-hub@auracite
```

## Publish for customers (public marketplace)

The AuraCite monorepo is **private**, so customers cannot `add` it. `/plugin marketplace add owner/repo`
also requires `.claude-plugin/marketplace.json` at the **repo root**, which the monorepo does not have.
Customer distribution therefore goes through a dedicated **public** repo whose root is this marketplace:

1. Create a public repo, e.g. `auracite/claude-plugins` (mirror the Codex `auracite/codex-plugins`
   naming).
2. Copy the contents of this directory to the repo root so `.claude-plugin/marketplace.json` sits at
   the root and `./auracite-agent-hub` resolves. Keep the monorepo copy as the source of truth and
   sync on release (a `git subtree split` of `plugins/auracite-claude-marketplace` or a CI mirror).
3. Customers then install with:

   ```text
   /plugin marketplace add auracite/claude-plugins
   /plugin install auracite-agent-hub@auracite
   ```

Alternatively, list each plugin with a `git-subdir` source pointing at this path inside a public
mirror — but the dedicated public marketplace repo is the simplest "as easy as Codex" path.

## Validate before publishing

```bash
claude plugin validate ./plugins/auracite-claude-marketplace
claude plugin validate ./plugins/auracite-claude-marketplace/auracite-agent-hub
```

Never commit an API key, token, or secret into this marketplace, the plugins, or their `.mcp.json`.
The connector reads the key from the `AURACITE_MCP_TOKEN` environment variable at runtime.
