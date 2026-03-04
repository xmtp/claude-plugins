# XMTP Claude Plugins

Official [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) for [XMTP](https://xmtp.org).

## What is a Plugin Marketplace?

A plugin marketplace is a catalog that distributes [Claude Code plugins](https://code.claude.com/docs/en/plugins) to your team and community. When someone adds this marketplace, they can browse and install any plugin listed here with a single command.

A plugin can contain any combination of:

- **Skills** — Markdown instructions that teach Claude how to perform specific tasks (e.g., using the XMTP CLI)
- **MCP servers** — Backend services that give Claude new tools, like searching documentation or querying APIs
- **Hooks** — Scripts that run automatically before or after Claude uses a tool (e.g., linting on file save)
- **Agents** — Specialized sub-agents for complex workflows (e.g., a security reviewer)
- **Commands** — Custom slash commands that trigger specific behaviors
- **LSP servers** — Language servers that provide code intelligence

Plugins are installed per-user and auto-update when the marketplace is refreshed.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add xmtp/claude-plugins

# Install a plugin
/plugin install xmtp-docs-mcp@xmtp-plugins
```

## Available Plugins

| Plugin | Description | Source |
|--------|-------------|--------|
| `xmtp-docs-mcp` | MCP server that exposes XMTP documentation as searchable tools | [xmtp/xmtp-docs-mcp](https://github.com/xmtp/xmtp-docs-mcp) |
| `xmtp-cli` | Skill for working with XMTP messaging via the xmtp CLI tool | Local |
| `xmtp-agent-sdk` | Skill for building XMTP messaging agents using @xmtp/agent-sdk | Local |

## Adding a Plugin

There are two ways to add a plugin to this marketplace. Both are configured in `.claude-plugin/marketplace.json`.

### Option A: Reference an external repo

If your plugin already lives in its own GitHub repo, point to it with a GitHub source. Use `strict: false` so the marketplace entry defines the plugin config (skills, MCP servers, hooks, etc.) without needing changes in the source repo.

```json
{
  "name": "my-plugin",
  "source": {
    "source": "github",
    "repo": "xmtp/my-plugin-repo"
  },
  "description": "What the plugin does",
  "version": "1.0.0",
  "strict": false,
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/dist/server.js"]
    }
  }
}
```

**When to use this:** The plugin has its own release cycle, CI, or is maintained by a different team. The marketplace just catalogs it.

### Option B: Add plugin code to this repo

Put your plugin under `plugins/` and use a relative path source. The plugin needs a `.claude-plugin/plugin.json` manifest and its content (skills, hooks, agents, etc.) in the standard directory structure.

```
plugins/
  my-plugin/
    .claude-plugin/
      plugin.json
    skills/
      my-skill/
        SKILL.md
```

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin",
  "description": "What the plugin does",
  "version": "1.0.0"
}
```

**When to use this:** Lightweight plugins (skills, hooks) that don't need their own repo, or plugins you want to iterate on quickly.

For the full plugin spec, see the [Claude Code plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces).

## License

MIT
