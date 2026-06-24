---
name: eucalyptus
description: "Use when working with Eucalyptus to turn Excel workbooks into API-backed calculation models through the Eucalyptus MCP server: list or inspect models and versions, upload Excel workbook versions, compile IO specs, run playground calculations, set API destination defaults, or read execution logs and metrics. Do not use for deleting models or versions, token/user administration, arbitrary REST/S3/Lambda calls, or unrelated Excel debugging."
---

# Eucalyptus

Use the installed `eucalyptus` MCP server as the only authority for Eucalyptus model state. The server's current tool names use the `calc.*` namespace. If the MCP tools are unavailable, tell the user the Eucalyptus plugin or MCP authentication is not configured and stop before inventing REST calls.

## Workflow

Start with the narrowest read path:

1. List models with `calc.models.list` only when the user did not name a model.
2. Fetch one model with `calc.models.get`.
3. List versions with `calc.versions.list`.
4. Fetch a selected version with `calc.versions.get` before compile, playground, or default changes.

For Excel workbook uploads, choose the reference by host:

- Generic local file or non-ChatGPT client: read `references/workflows.md`.
- ChatGPT Apps attached file: read `references/chatgpt-apps.md`.

For writes, state the intended model, version, and destination before calling the mutating tool. Never offer deletion; deletion is not available through this MCP server.

## Tool Map

- Models: `calc.models.list`, `calc.models.get`, `calc.models.create`, `calc.models.update_info`.
- Versions: `calc.versions.list`, `calc.versions.get`, `calc.versions.prepare_upload`, `calc.versions.confirm_upload`, `calc.versions.upload_file`, `calc.versions.compile`.
- Execution: `calc.versions.execute_playground`.
- Defaults: `calc.destinations.list`, `calc.versions.set_default_for_destination`.
- Observability: `calc.logs.search`, `calc.metrics.get`.

## References

Read `references/workflows.md` before upload, compile, playground, or default workflows.
Read `references/chatgpt-apps.md` only when the user provides a ChatGPT Apps attached workbook file.
Read `references/security.md` before any write, auth troubleshooting, or permission error.
