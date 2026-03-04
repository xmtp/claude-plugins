# CLAUDE.md

## What This Repo Is

This is a Claude Code plugin marketplace for the XMTP organization. It distributes plugins that help developers build with XMTP.

The marketplace name is `xmtp-plugins`. Users add it with `/plugin marketplace add xmtp/claude-plugins`.

## Repository Structure

```
.claude-plugin/marketplace.json   # The marketplace catalog — lists all plugins
plugins/                           # In-repo plugins (relative path source)
  xmtp-cli/                       # Example: skill plugin hosted locally
```

## How Plugins Are Sourced

Each plugin in `marketplace.json` has its own `source`. Two patterns are used:

- **External GitHub repo** — `"source": { "source": "github", "repo": "xmtp/..." }` with `strict: false`. The marketplace entry defines the plugin config. No changes needed in the source repo.
- **Relative path (in-repo)** — `"source": "./plugins/my-plugin"`. Plugin code lives in this repo under `plugins/`. Needs a `.claude-plugin/plugin.json` manifest.

## Current Plugins

- `xmtp-docs-mcp` — MCP server for searching XMTP docs. External source: `xmtp/xmtp-docs-mcp`. Uses `strict: false` so the marketplace defines the `mcpServers` config.
- `xmtp-cli` — Skill teaching Claude how to use the XMTP CLI. Lives in-repo at `plugins/xmtp-cli/`.

## Adding a New Plugin

1. Add an entry to `.claude-plugin/marketplace.json` in the `plugins` array
2. If in-repo: create `plugins/<name>/.claude-plugin/plugin.json` and add plugin content (skills, hooks, agents, etc.)
3. If external: set `strict: false` and define the plugin config (mcpServers, hooks, etc.) in the marketplace entry
4. Validate with `claude plugin validate .`
5. Update the Available Plugins table in `README.md`

## Conventions

- Marketplace validation: always run `claude plugin validate .` before pushing
- Plugin names use kebab-case
- In-repo plugins go under `plugins/`
- Use `${CLAUDE_PLUGIN_ROOT}` in MCP server args to reference files within the plugin's install directory
