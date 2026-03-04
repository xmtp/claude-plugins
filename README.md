# XMTP Claude Plugins

Official Claude Code plugin marketplace for [XMTP](https://xmtp.org).

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

### xmtp-docs-mcp

Provides two MCP tools:
- `search_xmtp_docs` — Search XMTP documentation by keyword
- `get_xmtp_doc_chunk` — Retrieve full content of a documentation section

Once installed, the MCP server is automatically registered — no manual `claude mcp add` needed.

### xmtp-cli

Teaches Claude how to use the `xmtp` CLI for sending messages, managing conversations, groups, consent, and identity operations.

Install:
```bash
/plugin install xmtp-cli@xmtp-plugins
```

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

### Which should I pick?

| | External repo | In this repo |
|---|---|---|
| Plugin has its own repo already | Yes | No |
| Plugin is just a skill or hook | No | Yes |
| Plugin needs its own CI/releases | Yes | No |
| Quick iteration with team | Either | Yes |
| MCP server with build step | Yes | Either |

For the full plugin spec, see the [Claude Code plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces).

## License

MIT
