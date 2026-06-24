# ChatGPT Apps Workbook Upload

Use this workflow only when the host provides a ChatGPT Apps file parameter for an attached Excel workbook. Codex, Claude Code, and other local-file clients should use `workflows.md` instead.

## Sequence

1. Confirm the user wants to turn the attached `.xlsx` or `.xlsb` workbook into a new API model version under the target `model_id`.
2. Call `calc.versions.upload_file` with `model_id` and the host-provided `workbook_file` parameter.
3. Call `calc.versions.compile` with the returned draft `version`, `inputs`, and `outputs`.
4. Poll `calc.versions.get` until compile succeeds or fails.
5. Set the destination default only when the user explicitly asks.

## File Parameter

The MCP tool advertises `_meta["openai/fileParams"]` for `workbook_file`. Use the host-provided file object; do not download or re-upload the file manually unless the host cannot pass file parameters.

## Boundaries

Do not use this flow for local paths, public URLs, arbitrary S3 keys, or non-ChatGPT MCP clients. If the file parameter is unavailable, switch to the generic prepare/upload/confirm workflow in `workflows.md` only when the environment can perform the direct PUT.
