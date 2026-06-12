MCPmarket plugin for Claude Code.

## Install

Install via Claude's plugin marketplace:

```
claude plugin marketplace add knoxgraeme/mcpmarket-plugin
```

Or download a pre-configured ZIP from https://app.mcpmarket.com.

## Local setup (this repo)

This plugin is registered as a project-scoped marketplace in
`../.claude/settings.json`. `.mcp.json` is gitignored because it contains
your personal MCPmarket API token. To activate it locally:

```
cp .mcp.json.example .mcp.json
```

Then edit `.mcp.json` and replace the placeholder with your real
`sk_user_...` token from https://app.mcpmarket.com (Toolkit settings).

## Layout

- `.claude-plugin/plugin.json` — plugin manifest
- `.mcp.json` — MCP server config (toolkit URL + API token)
- `hooks/hooks.json` — `SessionStart` hook
- `hook-shim.sh` — exports `MCPMARKET_*` env vars then runs `shared/sync.sh`

The shared sync logic lives in `shared/sync.sh`.
