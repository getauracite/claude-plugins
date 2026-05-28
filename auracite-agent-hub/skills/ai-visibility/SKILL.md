---
description: Work with AuraCite AI-visibility (GEO) data in Claude Code. Use when the user asks how their brand is mentioned, ranked, cited, or recommended by AI engines (ChatGPT, Gemini, Perplexity), wants their GEO visibility score, share-of-voice, competitor comparison, or trend data — backed by the AuraCite MCP connector.
---

# AuraCite AI Visibility (GEO)

AuraCite tracks how AI engines (ChatGPT, Gemini, Perplexity, …) mention, rank, cite, and recommend
brands — "Generative Engine Optimization" (GEO). This skill helps you answer visibility questions
using the AuraCite MCP connector (server name `auracite`).

## Prerequisites

The `auracite` MCP server must be connected. It needs an AuraCite API key in the
`AURACITE_MCP_TOKEN` environment variable. Get a read-only key (`mcp:read`) from the AuraCite app
under **API Keys** (admin-provisioned during the pilot) — that scope is enough for everything in
this skill. Check the connection with `/mcp` — if it isn't listed, see the plugin README.

## Tools available with a read-only key

| Tool | Use it for |
| --- | --- |
| `list_brands` | List the brands tracked in the project |
| `get_dashboard_stats` | Mentions / citations / sentiment summary (period `7d`/`30d`/`90d`) |
| `get_mentions` | Brand mentions detected in AI answers |
| `get_citations` | Direct citations/links a brand earned in AI answers |
| `list_prompts` | The tracked GEO prompts (the questions AuraCite asks the engines) |
| `list_competitors` | Auto-detected competitor brands |
| `get_visibility_score` | Composite 0–100 GEO score with letter grade |
| `get_share_of_voice` | Share-of-voice % across AI engines |
| `get_trend_data` | Time-series for mentions/citations/sentiment |
| `get_ai_engine_breakdown` | Per-engine breakdown (ChatGPT/Gemini/Perplexity/…) |
| `compare_brands` | Two brands side-by-side |

`tenant_id` and `project_id` are injected server-side from the API key — you cannot read another
tenant's data, and you don't need to pass them. Most tools need a `brand_id` or `project_id` you can
get from `list_brands`.

## Good working pattern

1. `list_brands` to find the brand/project the user means.
2. `get_dashboard_stats` or `get_visibility_score` for the headline picture.
3. Drill in with `get_mentions` / `get_citations` / `get_ai_engine_breakdown` / `compare_brands`.
4. When the user asks "what should we do about it," reason from the evidence (which engines, which
   prompts, which competitors are winning citations) — don't invent numbers the tools didn't return.

## Safety / scope

- This skill is **read-only**. Scan/ingestion, brand/prompt creation, and credit-spending tools are
  hidden from read-only keys and require an `mcp:write`/`mcp:spend` key plus an explicit AuraCite
  approval flow. Don't tell the user a write happened unless a write tool actually returned success.
- Cost-bearing actions (new scans, LLM/provider calls) consume AuraCite credits — never trigger them
  speculatively; confirm with the user first and report the actual cost the tool returns.
