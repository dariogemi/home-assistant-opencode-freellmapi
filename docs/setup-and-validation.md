# Safe setup and end-to-end validation

This is a controlled runbook, not a copy-and-paste installer. Provider schemas, CLI commands, add-on options, and UI paths change across releases. Verify them in the versions actually installed before using them.

## 0. Scope and safety

In scope:

- inspect the Home Assistant OpenCode add-on/runtime;
- compare non-secret provider details with a known-working client;
- verify network reachability and API compatibility;
- configure the minimum required provider settings through a supported mechanism;
- perform one low-cost functional test;
- document evidence and rollback.

Out of scope:

- reinstalling or upgrading Home Assistant/OpenCode as a first response;
- changing global DNS, gateway, firewall, VPN, or router settings;
- exposing a private model endpoint to the Internet;
- changing unrelated add-ons, cameras, voice pipelines, automations, or databases;
- editing internal storage or undocumented state files;
- publishing credentials or private configuration.

## 1. Record the baseline (read-only)

Record:

- date/time and timezone;
- Home Assistant version;
- add-on version and state;
- OpenCode version from inside the target runtime;
- current provider/model selection, with secrets redacted;
- recent relevant logs, sanitized;
- current network path and the location from which reachability is tested;
- whether another process or agent may write the same configuration.

Do not restart a healthy add-on just to collect information. Do not dump complete environment variables or configuration objects: they may contain credentials.

## 2. Inspect the known-working client

If a workstation OpenCode installation already uses FreeLLMAPI successfully, use it only as a comparison source.

Collect non-secret facts:

- OpenCode version;
- provider identifier and protocol type;
- base endpoint (remove credentials and private details before publication);
- exact model IDs that work;
- whether a routing alias (for example, `auto`) is supported;
- the minimal request shape and whether streaming is used.

Never copy API keys, tokens, cookies, authorization headers, or credential files. Do not modify the working workstation configuration to make the Home Assistant target match it.

## 3. Verify the endpoint contract

Use the provider's current documentation or an explicitly supported health/model-list operation.

Confirm:

- base URL and API path;
- supported HTTP methods and routes;
- authentication method;
- request and response schema;
- streaming support if the client requires it;
- exact model IDs and alias behavior;
- rate limits, quotas, and context limits.

Do not assume that `/v1/models`, `/chat/completions`, or any other route exists until the provider documentation or a safe read-only probe confirms it. Do not infer model IDs from display names.

## 4. Verify reachability from the target runtime

Test from the environment that runs OpenCode in Home Assistant, not merely from a workstation or the Home Assistant host.

Classify the result:

- DNS/name resolution;
- route/connectivity;
- connection refused;
- timeout;
- TLS/certificate;
- authentication;
- API/schema response.

Do not solve a reachability problem by disabling a firewall, opening a public port, or changing global network configuration. If the only available fix is a global or security-sensitive network change, stop and document the required decision.

## 5. Configure the provider

Before writing:

1. Verify the installed OpenCode version and supported configuration schema.
2. Verify the documented UI or API/CLI mechanism and its persistence behavior.
3. Create a recoverable backup outside Git.
4. Confirm the backup is readable without displaying its contents.
5. Prepare a minimal diff that changes only the required provider fields.
6. Confirm rollback steps.

Only then configure the provider through the supported mechanism.

The conceptual values are:

| Setting | Meaning | Validation |
|---|---|---|
| Provider ID | Stable local identifier | Must meet the installed version's naming rules |
| Provider type | API protocol/client adapter | Must match the actual FreeLLMAPI contract |
| Base URL | Provider endpoint | Reachable from the OpenCode runtime |
| Authentication | Credential mechanism | Documented and stored securely |
| Model ID | Exact provider model identifier | Confirmed in the provider catalog |
| Alias | Optional routing shortcut | Use only if provider and client both support it |

Do not paste a generic JSON example into a production add-on without checking its schema.

## 6. Functional proof

After configuration:

1. Confirm the provider appears through the supported read-only mechanism.
2. Confirm the selected model ID is available.
3. Send one harmless, low-cost request from the OpenCode runtime in the Home Assistant environment, such as asking for the exact response `OK`.
4. Verify evidence that the request used the intended provider and model, rather than a default provider or fallback.
5. Record duration, result, sanitized error details, and relevant timestamps.
6. Observe add-on health and logs for a bounded period (up to five minutes; no continuous polling).

A provider appearing in a settings screen is not functional proof. A successful request from a workstation is not proof of the Home Assistant runtime path.

## 7. Persistence and rollback

Verify persistence using the supported configuration read path. Restart only if restart is required to validate persistence, the impact is understood, and a recoverable backup exists. Never restart Home Assistant itself as a routine provider test.

If the change breaks startup or requests:

- stop further experiments;
- restore only the change introduced by this procedure;
- do not overwrite unrelated changes made after the backup;
- verify the add-on and service recover;
- record the result as failed/partial, not successful.

## 8. Acceptance criteria

Mark the integration **PASS** only when all are true:

- the target add-on/runtime is healthy;
- the provider is configured through a supported and persistent mechanism;
- the exact model ID is verified and available;
- a real request initiated from OpenCode in the Home Assistant environment succeeds;
- evidence identifies the intended provider/model;
- the network path is verified from the runtime;
- no unexpected restarts, loops, or runaway retries occur during observation;
- rollback is known and available;
- no secrets were exposed and no unrelated component was changed.

Use **PARTIAL** when only some checks pass, **NO-GO** when safe execution is blocked, and **UNCERTAIN** when evidence is contradictory or insufficient.
