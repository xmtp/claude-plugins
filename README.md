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

## Contributing

To add a plugin to this marketplace, see the [Claude Code plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces).

## License

MIT
