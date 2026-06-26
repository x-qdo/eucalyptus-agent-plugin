---
name: eucalyptus
description: "Use when working with Eucalyptus to turn Excel workbooks into API-backed calculation models through the Eucalyptus MCP server: list or inspect models and versions, choose locale/regional settings, upload Excel workbook versions, discover IN_/OUT_ cells, compile IO specs, run playground calculations, create/update/run saved tests, inspect test results/history, set API destination defaults, or read execution logs and metrics. Do not use for deleting models, versions, or tests, token/user administration, arbitrary REST/S3/Lambda calls, or unrelated Excel debugging."
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

Before creating a model or uploading a replacement workbook version, confirm the model locale/regional settings. Locale controls text-date, decimal, grouping, and currency interpretation and must be set before upload/compile workflows when it needs to change.

After workbook upload, call `calc.versions.discover_io` and ask the user to confirm the detected cells. `calc.versions.compile` accepts confirmed `discover_io` cells directly; do not reduce them to string name arrays.

For writes, state the intended model, version, and destination before calling the mutating tool. Never offer deletion; deletion is not available through this MCP server.

Before changing a destination default or explaining where execution/logs will run, read `references/destinations.md`.

## Tool Map

- Models: `calc.models.list`, `calc.models.get`, `calc.models.create`, `calc.models.update_info`.
- Versions: `calc.versions.list`, `calc.versions.get`, `calc.versions.prepare_upload`, `calc.versions.confirm_upload`, `calc.versions.upload_file`, `calc.versions.discover_io`, `calc.versions.compile`.
- Execution: `calc.versions.execute_playground`.
- Tests: `calc.tests.list`, `calc.tests.get`, `calc.tests.create`, `calc.tests.update`, `calc.tests.run`, `calc.tests.run_model_version`, `calc.tests.results`, `calc.tests.version_results`.
- Defaults: `calc.destinations.list`, `calc.versions.set_default_for_destination`.
- Observability: `calc.logs.search`, `calc.metrics.get`.

Execution logs are destination-scoped and are ingested asynchronously. Playground executions run in the account primary/default runtime destination when one is configured, not automatically in SaaS; look for playground logs under that destination and allow up to 3 minutes for ingestion.

## References

Read `references/workflows.md` before upload, compile, playground, or default workflows.
Read `references/destinations.md` before changing destination defaults, explaining SaaS vs delegated defaults, or preparing automated engine-deploy test coverage.
Read `references/chatgpt-apps.md` only when the user provides a ChatGPT Apps attached workbook file.
Read `references/security.md` before any write, auth troubleshooting, or permission error.
