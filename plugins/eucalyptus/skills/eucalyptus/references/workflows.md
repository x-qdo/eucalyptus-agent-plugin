# Eucalyptus MCP Workflows

## Inspect Models

Use this sequence for discovery:

1. `calc.models.list`
2. `calc.models.get` with `model_id`
3. `calc.versions.list` with `model_id`
4. `calc.versions.get` with `model_id` and `version`

Return compact summaries by default: model id, alias, default version, version number, status, compile status, created time, and destination default status. Do not paste full IO specs unless the user asks.

## Generic Excel Workbook Upload

Use this for Codex, Claude Code, local files on macOS/Linux/Windows, and other generic MCP clients. For ChatGPT Apps attached workbook files, stop here and read `chatgpt-apps.md` instead.

1. Confirm the user wants to turn the named `.xlsx` or `.xlsb` file into a new API model version under the target `model_id`.
2. Call `calc.versions.prepare_upload` with `model_id` and `filename`.
3. Upload the local file bytes to the returned `upload_url` using exactly the returned `content_type` if present. Do not send the workbook anywhere else.
4. Call `calc.versions.confirm_upload` with the returned `model_id`, `version`, and `s3_key`.
5. Call `calc.versions.compile` with the confirmed `model_id`, `version`, `inputs`, and `outputs`.
6. Poll `calc.versions.get` until the compile state is final.
7. If the user asked to promote it, call `calc.destinations.list`, choose the requested destination, then call `calc.versions.set_default_for_destination`.

If the environment cannot perform the direct PUT, stop after `prepare_upload` and tell the user the direct upload step is blocked.

## IO Specification

Compile expects `inputs` and `outputs` arrays that define the workbook cells exposed by the API. Prefer the workbook's known IO spec when the user provides one. If the user asks for auto-discovery and the server supports it through the compile schema in the current deployment, use it; otherwise ask for the IO spec instead of guessing.

## Playground Calculation

Use `calc.versions.execute_playground` only against a compiled version. First check the version with `calc.versions.get` if the compile state is unknown.

Inputs must be a JSON object keyed by model input names. Return outputs, calculation status, and any user-relevant warnings or errors. Do not create API tokens for playground execution.

## Destination Defaults

Use `calc.destinations.list` before setting a default unless the user gave a known destination id.

Call `calc.versions.set_default_for_destination` only after confirming:

- target `model_id`
- target `version`
- destination id, or SaaS/default destination when omitted

Setting a model version default is destination-scoped. It does not change the account primary destination.

## Logs And Metrics

Use `calc.logs.search` for raw executions and filtering by model, version, destination, source, status, month, or limit.

Use `calc.metrics.get` for dashboard, model, aggregate, cache, or hourly summaries. Ask for a month or date when the requested metric requires one.
