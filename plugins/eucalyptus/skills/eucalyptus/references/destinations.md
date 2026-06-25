# Destinations And Default Versions

Use this reference when a user asks to promote a version, inspect default versions, prepare automated engine-deploy validation, or explain where playground/API traffic will run.

## Concepts

- A destination is a runtime target. `saas` is the Eucalyptus-hosted runtime; delegated destinations are customer/runtime targets such as production or beta runtimes.
- A model version default is destination-scoped. Setting a default for one destination does not change defaults for other destinations.
- The account primary/default runtime destination is different from a model version default. It controls where playground execution and some upload/runtime operations are routed when a delegated runtime is configured.
- `calc.versions.set_default_for_destination` changes the model version default for one destination only. It does not configure, enable, disable, delete, or make a destination primary.

## When To Update Defaults

Update a destination default only when the user has confirmed that a compiled version should become the active version for that runtime destination.

Common cases:

- SaaS validation: set or keep the `saas` default when the model should participate in automated tests for a new Eucalyptus engine deploy. Engine-deploy auto tests run on the SaaS runtime and need a SaaS destination default; a delegated-only default does not make that model part of SaaS engine validation.
- Delegated runtime promotion: set a delegated destination default when the customer/runtime traffic for that destination should move to the new version.
- Staged rollout: promote SaaS and delegated destinations independently. Do not assume a version validated on one destination is defaulted on another.
- Repair: if API calls or tests are hitting an older version, inspect destination defaults before recompiling or uploading again.

Do not auto-promote after compile. Ask which destination should be updated unless the user explicitly named it.

## Safe Promotion Sequence

1. Call `calc.destinations.list`.
2. Identify the requested destination id. Use `saas` only when the user asks for SaaS, hosted runtime, or engine-deploy auto-test coverage.
3. Call `calc.versions.get` and confirm the target version is compiled.
4. Confirm the model id, version number, destination id, and reason for promotion.
5. Call `calc.versions.set_default_for_destination` with `model_id`, `version`, and `destination_id`.
6. Re-read the model or version list and summarize the updated destination default.

If the user says "make this live" or "promote it" without a destination, ask whether they mean `saas`, a delegated destination, or both.

## Automated Engine Deploy Tests

For engine-deploy auto tests, make sure each model that should be tested has:

- a compiled version compatible with the SaaS runtime;
- `saas` set to that version as the destination default;
- enough representative inputs/tests in the platform test suite or test harness.

If a customer production destination is primary, playground runs may execute there, but SaaS engine-deploy tests still need the `saas` default. Treat "playground succeeded" and "SaaS auto tests will cover this model" as separate checks.

## Logs And Metrics By Destination

Logs and metrics are destination-scoped. Playground execution uses the account primary/default runtime destination when it has a runtime Lambda configured, and falls back to SaaS only when no such primary destination is available.

When investigating execution logs:

- use `calc.destinations.list` to identify the destination;
- filter logs by that destination id;
- use `source: "playground"` for playground runs;
- allow up to 3 minutes for logs to appear.

Do not conclude that a playground run did not log just because `saas` logs are empty.
