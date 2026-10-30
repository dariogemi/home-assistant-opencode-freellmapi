# Home Assistant + OpenCode + FreeLLMAPI

A practical, security-first field guide for connecting **OpenCode** running alongside **Home Assistant** to models exposed by **FreeLLMAPI**.

This repository documents the integration approach, the checks required before configuration, operational validation, and safe troubleshooting. It deliberately separates reusable guidance from deployment-specific facts.

> **Status:** the integration procedure is documented, but this repository does not claim that a working end-to-end deployment has been independently demonstrated. See [implementation status](docs/implementation-status.md).

## What this project covers

- The roles and boundaries of Home Assistant, the OpenCode add-on/runtime, and FreeLLMAPI.
- How to verify network reachability and an OpenAI-compatible API contract before configuring a provider.
- Provider/model configuration principles, including why model IDs and aliases must be verified rather than guessed.
- A read-only-first deployment and validation workflow.
- Troubleshooting, rollback, stability controls, and secret-handling rules.

## Architecture at a glance

```text
Home Assistant
  automations / scripts / user requests
                |
                | supported integration or tool boundary
                v
OpenCode runtime (Home Assistant host/add-on)
  provider configuration + model selection
                |
                | API request (protocol must be verified)
                v
FreeLLMAPI
  model routing / selected model
                |
                v
       response to OpenCode
                |
                v
Home Assistant workflow / user
```

The exact invocation path between Home Assistant and OpenCode depends on the installed integration and versions. This diagram describes the logical roles; it is not a claim that every arrow is already configured.

## Start here

1. [Architecture and component responsibilities](docs/architecture.md)
2. [Safe setup and validation runbook](docs/setup-and-validation.md)
3. [Operations and troubleshooting](docs/operations-and-troubleshooting.md)
4. [Implementation status and evidence](docs/implementation-status.md)
5. [Security policy](SECURITY.md)

## Core principles

1. **Read before write.** Inspect versions, current configuration, logs, and network paths before changing anything.
2. **Verify the contract.** Confirm the installed OpenCode version, provider schema, API endpoint, authentication method, and model IDs from authoritative sources.
3. **One change at a time.** Back up first, make a minimal reversible change, then validate it.
4. **Prove the full path.** A reachable endpoint or visible provider is not proof. A real request must originate from the OpenCode runtime in the Home Assistant environment and use the intended provider/model.
5. **Keep secrets out of Git.** Never commit API keys, tokens, cookies, authorization headers, private configuration, logs containing credentials, or environment dumps.
6. **Do not disturb unrelated systems.** Changes to Home Assistant networking, DNS, firewalls, cameras, voice pipelines, or other add-ons are outside this project's scope unless separately designed and approved.

## Compatibility

There is no universal configuration snippet that is safe to copy across all OpenCode add-on releases. Before applying examples, verify:

- Home Assistant and add-on versions;
- the OpenCode version actually running inside the add-on;
- the supported provider configuration mechanism for that version;
- whether the target endpoint implements the expected API protocol;
- the exact model identifier and whether aliases such as `auto` are supported.

Treat all deployment-specific values as examples until verified in the target environment.

## Contributions

Prefer small, reviewable documentation changes. Label claims as **verified**, **reported**, **proposed**, or **unknown**. Never turn a plan into a success report without test evidence.
