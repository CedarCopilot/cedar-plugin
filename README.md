# Cedar Claude Plugin

Connects [Cedar](https://cedarcopilot.com) to Claude Code and Claude Cowork: Cedar's
tools via MCP, and conversation capture (via hooks) so what you do with Cedar through
Claude shows up in your Cedar chat history.

## Install

**Claude Cowork**: open the Marketplace section, search for "cedar", click Install.

**Claude Code**:
```
/plugin marketplace add CedarCopilot/cedar-plugin
/plugin install cedar@cedar-marketplace
```

## What's in here

- `.claude-plugin/marketplace.json` — the marketplace catalog (this repo is both the
  marketplace and the one plugin it lists).
- `cedar-plugin/` — the plugin itself:
  - `.mcp.json` — connects to Cedar's remote MCP server (OAuth-authenticated on first
    use).
  - `hooks/hooks.json` — `UserPromptSubmit`/`Stop`/`PostToolUse` hooks that forward your
    conversation and tool activity to Cedar over the same authenticated connection, so
    it's recorded in your Cedar chat history. No separate credential — these ride the
    same OAuth session as the MCP tools.

Cedar's own skills (drafting, CRM, brain, etc.) aren't bundled as native files here —
they're loaded on demand via the `load-skill` MCP tool, which always reflects Cedar's
current skill catalog with no separate copy to keep in sync.

## Local development

`cedar-plugin/.mcp.json`'s `url` defaults to Cedar's production MCP endpoint, but
supports Claude Code's `${VAR:-default}` environment variable expansion. To point a
local Claude Code session at a dev server (e.g. an ngrok tunnel) instead, without
editing (and risking committing a change to) the file itself:

```
export CEDAR_MCP_URL="https://your-tunnel.ngrok.app/mcp"
claude --plugin-dir /path/to/your/local/checkout/of/cedar-plugin/cedar-plugin
```

Unset the variable (or open a fresh shell) to fall back to the production URL. Never
commit a change to the literal default value in `.mcp.json` to point at a dev URL —
CI's dev-artifact guard will reject it, and it would be live for every real customer.

## Design doc

See `docs/design/mcp-observability-and-plugin.md` in the `cedar-mail` repo (private) for
the full design history and open decisions.
