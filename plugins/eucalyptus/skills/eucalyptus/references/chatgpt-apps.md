# ChatGPT Apps Workbook Upload

Use this workflow only when the host provides a ChatGPT Apps file parameter for an attached Excel workbook. Codex, Claude Code, and other local-file clients should use `workflows.md` instead.

## Sequence

1. Confirm the user wants to turn the attached `.xlsx` or `.xlsb` workbook into a new API model version under the target `model_id`.
2. Confirm the model locale before upload. If it must change for this workbook, call `calc.models.update_info` with `regional_settings` before `calc.versions.upload_file`; see `workflows.md` for locale details.
3. Call `calc.versions.upload_file` with `model_id` and the host-provided `workbook_file` parameter.
4. Call `calc.versions.discover_io` with the returned draft `version`.
5. Show the detected `IN_*` and `OUT_*` cells to the user for confirmation.
6. Convert confirmed `discovered_io` cells to compile `inputs` and `outputs`, then call `calc.versions.compile`.
7. Poll `calc.versions.get` until compile succeeds or fails.
8. Set the destination default only when the user explicitly asks; read `destinations.md` first.
9. When checking logs after playground execution, search the account primary/default runtime destination first. Playground logs are not necessarily under `saas`.

## File Parameter

The MCP tool advertises `_meta["openai/fileParams"]` for `workbook_file`. Use the host-provided file object; do not download or re-upload the file manually unless the host cannot pass file parameters. After upload, discovery is still explicit through `calc.versions.discover_io`; `compile` should receive user-confirmed IO cells.

## Boundaries

Do not use this flow for local paths, public URLs, arbitrary S3 keys, or non-ChatGPT MCP clients. If the file parameter is unavailable, switch to the generic prepare/upload/confirm workflow in `workflows.md` only when the environment can perform the direct PUT.
