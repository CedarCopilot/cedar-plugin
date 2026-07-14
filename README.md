# Cedar Claude Plugin

Connects [Cedar](https://cedarcopilot.com) to Claude Code: Cedar's tools via MCP, and
conversation capture (via hooks) so what you do with Cedar through Claude shows up in
your Cedar chat history.

## Install

**Claude Code** (full plugin — tools + conversation capture):
```
/plugin marketplace add CedarCopilot/cedar-plugin
/plugin install cedar@cedar-marketplace
```

**Claude Cowork** (tools only, no conversation capture — see below):
Cowork's plugin-marketplace sync currently rejects this plugin outright. Its
`hooks/hooks.json` uses the `mcp_tool` hook type (the only mechanism that lets a hook
call an MCP tool directly — required for conversation capture), but Cowork's
server-side marketplace-sync validator only accepts `command`/`prompt`/`agent` hook
types. This is a confirmed, deliberate Anthropic-side restriction, not a bug in this
repo — see [anthropics/claude-code#35653](https://github.com/anthropics/claude-code/issues/35653)
(closed `NOT_PLANNED`) for the identical restriction on HTTP hooks. Verified live via
`~/Library/Logs/Claude/claude.ai-web.log`'s `MARKETPLACE_ERROR:REMOTE_SYNC_FAILED`
payload, and by testing a hooks-free build of this exact repo (synced successfully with
hooks removed, failed identically with them present).

Until Anthropic lifts this restriction, add Cedar as a **custom connector** in Cowork
instead of installing this plugin: **Customize → Connectors → Add custom connector**,
URL `https://api.mail.cedarcopilot.com/mcp`. This gets you the same MCP tools with no
plugin involved — but no conversation capture, since that's entirely hook-driven.

## Testing (Claude Code — most controllable path, verified working)

After installing (above), this kicks off Cedar's OAuth consent flow the first time it
needs to call a tool. Then:

1. `/mcp` — confirm `cedar-mcp` shows connected, and its tool list includes the family
   tools (`mail`, `conversation`, `brain`, etc.) plus `load-skill`, `record-message`, and
   `record-tool-call`.
2. Send any message and let the turn finish.
3. Check Cedar's `chat_threads`/`chat_messages` tables (or the Cedar chat UI, if there's
   a view for it) for a new thread keyed by this Claude Code session's id, containing
   your prompt and the assistant's reply.
4. Ask it to do something with an actual Cedar tool (e.g. "what's in my inbox") and
   confirm a `type: 'tool-call'` row shows up too.
5. Confirm the thread/messages are attributed to your real Cedar user, not anonymous.

To test against a dev server instead of production, set `CEDAR_MCP_URL` first — see
"Local development" below.

**Note on scope**: the `PostToolUse` hook fires for *every* tool call in the Claude Code
session, not just Cedar's. If other MCP servers are connected in the same session,
their tool calls get forwarded to `record-tool-call` too — Cedar ends up with a full
activity log of that session, not just the Cedar-specific parts. This is a deliberate
scope decision (see the design doc), not a bug — worth being aware of when testing.

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
