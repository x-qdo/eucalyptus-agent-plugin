# Eucalyptus MCP Workflows

## Inspect Models

Use this sequence for discovery:

1. `calc.models.list`
2. `calc.models.get` with `model_id`
3. `calc.versions.list` with `model_id`
4. `calc.versions.get` with `model_id` and `version`

Return compact summaries by default: model id, alias, default version, version number, status, compile status, created time, and destination default status. Do not paste full IO specs unless the user asks.

## Locale And Regional Settings

Locale is part of the model's `regional_settings` and is copied into versions for workbook interpretation. Decide it before creating a model or before uploading/compiling the next workbook version.

Locale affects:

- ambiguous text dates: `03/01/2026` is March 1 with `en-us` and January 3 with DMY locales.
- text numbers: decimal and grouping separators differ, for example `1,234.57`, `1 234,57`, and `1.234,57`.
- currency/text formatting in formulas that parse or format localized strings.

Supported locale values: `en-us`, `en-gb`, `en-ie`, `et-ee`, `pl-pl`, `en-au`, `da-dk`, `de-de`, `de-at`. `runner` is a legacy runtime locale; do not choose it for new models unless the user explicitly needs legacy behavior.

When creating a model, pass `regional_settings` to `calc.models.create`, for example:

```json
{
  "alias": "rating-model",
  "description": "Fleet pricing workbook",
  "regional_settings": { "locale": "en-gb" }
}
```

For fixed locales, Eucalyptus derives `date_order` from `locale`; use `en-us` for MDY and most other supported locales for DMY. If the user says the workbook was authored in US Excel, prefer `en-us`. If the workbook uses comma decimals or European grouping, choose the matching locale before upload.

To change locale for the next version of an existing model, call `calc.models.update_info` with `regional_settings` before `calc.versions.prepare_upload` or `calc.versions.compile`. Existing compiled versions keep the locale they were compiled with; changing the model locale affects future uploaded/compiled versions, not old bytecode. After a locale change, rediscover IO if the workbook has localized text values and ask the user to confirm before compile.

## Generic Excel Workbook Upload

Use this for Codex, Claude Code, local files on macOS/Linux/Windows, and other generic MCP clients. For ChatGPT Apps attached workbook files, stop here and read `chatgpt-apps.md` instead.

1. Confirm the user wants to turn the named `.xlsx` or `.xlsb` file into a new API model version under the target `model_id`.
2. Confirm the model locale. If it must change for this upload, call `calc.models.update_info` before preparing the upload.
3. Call `calc.versions.prepare_upload` with `model_id` and `filename`.
4. Upload the local file bytes to the returned `upload_url` using exactly the returned `content_type` if present. Do not send the workbook anywhere else.
5. Call `calc.versions.confirm_upload` with the returned `model_id`, `version`, and `s3_key`.
6. Call `calc.versions.discover_io` with the confirmed `model_id` and `version`.
7. Show the discovered input/output names and value addresses to the user for confirmation before compile. Keep the summary compact.
8. Convert confirmed `discovered_io.inputs` and `discovered_io.outputs` to the `inputs` and `outputs` arrays for `calc.versions.compile`.
9. Poll `calc.versions.get` until the compile state is final.
10. If the user asked to promote it, call `calc.destinations.list`, choose the requested destination, then call `calc.versions.set_default_for_destination`.

If the environment cannot perform the direct PUT, stop after `prepare_upload` and tell the user the direct upload step is blocked.

## IO Specification

Compile expects `inputs` and `outputs` arrays that define the workbook cells exposed by the API. Prefer the workbook's known IO spec when the user provides one. Otherwise call `calc.versions.discover_io` after upload confirmation and ask the user to confirm the detected `IN_*` and `OUT_*` cells before compiling. Do not make `compile` silently discover IO.

Map each confirmed discovered cell to compile IO shape as: `name` = discovered `name`, `address` = discovered `value_address`, `sheet` = discovered `sheet`, and `default_value` = discovered `current_value` when present. Omit or edit cells the user rejects before calling `calc.versions.compile`.

## Playground Calculation

Use `calc.versions.execute_playground` only against a compiled version. First check the version with `calc.versions.get` if the compile state is unknown.

Inputs must be a JSON object keyed by model input names. Return outputs, calculation status, and any user-relevant warnings or errors. Do not create API tokens for playground execution.

Playground execution uses the account primary/default runtime destination when it has a runtime Lambda configured. It falls back to the SaaS runtime only when no such primary destination is available. This means playground logs and metrics may appear under a delegated destination id, not under `saas`.

## Saved Tests

Saved tests are regression cases attached to a model. They are separate from playground calculations: playground is ad hoc execution; saved tests persist inputs and expected outputs, store run history, and can be rerun against selected compiled versions.

Use this sequence:

1. `calc.tests.list` with `model_id` to inspect compact test metadata. The list result intentionally returns input/output names, not full values.
2. `calc.tests.get` before editing or when exact saved inputs/expected outputs are needed.
3. `calc.tests.create` with `model_id`, `name`, optional `description`, `inputs`, and `expected_outputs` to add a test.
4. `calc.tests.update` with `model_id`, `test_id`, and only the fields to replace.
5. `calc.tests.run` with `model_id`, `test_id`, and target compiled `version` to run one saved test.
6. `calc.tests.results` to inspect recent execution history for one saved test.
7. `calc.tests.version_results` to inspect recent saved-test results for one model version.

Use `calc.tests.run_model_version` only when the user asks to run all saved tests for a model against one compiled version. Summarize pass/fail counts first, then expand failed results.

Do not delete tests through MCP. If the user asks to remove a test, say deletion is not exposed through the Eucalyptus MCP server.

## Destination Defaults

For detailed guidance on destination-scoped defaults, primary/default runtime destinations, and SaaS defaults for engine-deploy auto tests, read `destinations.md`.

Use `calc.destinations.list` before setting a default unless the user gave a known destination id.

Call `calc.versions.set_default_for_destination` only after confirming:

- target `model_id`
- target `version`
- destination id, or SaaS/default destination when omitted

Setting a model version default is destination-scoped. It does not change the account primary destination.

## Logs And Metrics

Use `calc.logs.search` for raw executions and filtering by model, version, destination, source, status, month, or limit.

Logs and metrics are destination-scoped. Before investigating a playground run, identify the account primary/default destination from `calc.destinations.list`; search that destination first, and do not treat an empty `saas` result as evidence that the playground run did not log. Use `source: "playground"` plus the destination filter when the user asks for logs from a just-run playground calculation.

Execution logs are asynchronous. Recent playground or API executions may take up to 3 minutes to appear in `calc.logs.search`; if a just-run execution is missing, say it may still be ingesting and retry before concluding there are no logs.

Use `calc.metrics.get` for dashboard, model, aggregate, cache, or hourly summaries. Ask for a month or date when the requested metric requires one.
