# Eucalyptus Agent Plugin

Public plugin marketplace for Eucalyptus workflows in Codex and Claude Code. Eucalyptus turns Excel workbooks into API-backed calculation models.

The plugin bundles:

- an `eucalyptus` skill with the Excel-to-API workflow guide
- an Eucalyptus MCP server configuration
- Codex and Claude Code plugin manifests
- Codex and Claude marketplace catalogs

The MCP endpoint is configured in `plugins/eucalyptus/.mcp.json`.

## Install In Codex

Add the repo marketplace:

```bash
codex plugin marketplace add x-qdo/eucalyptus-agent-plugin --ref main
```

Then open `/plugins`, select the `QDO Eucalyptus` marketplace, install `Eucalyptus`, and start a new thread.

Example:

```text
Use $eucalyptus to list my Excel-backed API models and summarize compiled versions.
```

## Install In Claude Code

For local validation from this checkout:

```bash
claude --plugin-dir ./plugins/eucalyptus
```

For marketplace validation after pushing:

```text
/plugin marketplace add x-qdo/eucalyptus-agent-plugin
/plugin install eucalyptus@qdo-eucalyptus
```

Example:

```text
/eucalyptus:eucalyptus list my Excel-backed API models and summarize ids, aliases, and defaults
```
