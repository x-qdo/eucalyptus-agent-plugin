# Eucalyptus MCP Security Rules

## Authorization

The Eucalyptus MCP server is OAuth-protected with Cognito access tokens. Use the MCP client's normal OAuth flow. Do not ask users to paste bearer tokens into chat.

If a tool returns an insufficient-scope or authentication challenge, report the missing capability and stop. Do not retry with unrelated tools or raw HTTP endpoints.

## Data Minimization

Do not expose full workbook contents, full IO specs, logs, or metrics by default. Summarize and ask before returning large payloads.

Do not upload a file unless the user explicitly asked to upload that exact Excel workbook to a target model.

Use direct HTTP upload only for the presigned `upload_url` returned by `calc.versions.prepare_upload`. Do not use arbitrary S3, Lambda, API Gateway, or management REST calls.

## Allowed Operations

Allowed:

- list and inspect models, versions, destinations, logs, and metrics
- create or update model metadata
- upload Excel workbook versions
- compile versions
- run playground calculations
- set a compiled version as the default for one destination

Not available:

- delete models
- delete versions
- create, reveal, or revoke API tokens
- manage users or Cognito groups
- configure, disable, delete, or test destinations
- arbitrary REST, S3, or Lambda operations

When a user asks for a forbidden operation, state that the installed MCP server intentionally does not expose that action and offer the closest safe read-only alternative.
