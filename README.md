# Cedar Claude Plugin

Connects [Cedar](https://cedarcopilot.com) to Claude: your email, meetings, Slack, and
CRM tools, right from your Claude conversation.

## Install

**Claude Code**:
```
/plugin marketplace add CedarCopilot/cedar-plugin
/plugin install cedar@cedar-marketplace
/mcp
```
Run `/mcp` after installing to connect Cedar's MCP server — without it, Cedar's tools
won't be available even though the plugin shows as installed.

**Claude Cowork**: add Cedar as a custom connector — **Customize → Connectors → Add
custom connector**, using URL `https://api.mail.cedarcopilot.com/mcp`.

The first time Cedar needs to access your account, you'll be prompted to sign in and
authorize access.

## Support

Questions or issues? Visit [cedarcopilot.com](https://cedarcopilot.com).
