# Implementation status and evidence

**Last reviewed:** 2026-10-30  
**Repository:** `dariogemi/home-assistant-opencode-freellmapi`  
**Evidence policy:** a documented plan is not an executed test; a visible provider is not proof of a successful model request.

## Current public-documentation status

The repository is being established as a reusable integration guide. Its documentation describes the target architecture and a safe validation method. It does not claim a verified end-to-end deployment.

## Deployment evidence snapshot

The working notes for the Home Assistant OpenCode recovery effort were updated on 2026-10-27. They describe a recovery/verification mission whose acceptance criterion is a real request originating from the OpenCode runtime in Home Assistant and answered by the intended FreeLLMAPI provider/model.

The available record at that point includes:

- a FreeLLMAPI provider identifier, `freellmapi_local`, being added to managed provider configuration;
- a subsequent add-on restart;
- a running add-on/OpenCode V2 environment reported during the recovery work;
- explicit outstanding verification of the provider API and a real model request from the Home Assistant OpenCode runtime.

The available evidence does **not** establish that the provider API was successfully read, that the model alias was available, or that an end-to-end request succeeded. Therefore the deployment status in this public guide remains **NOT YET PROVEN**.

## What would change this status to PASS

Publish a new dated execution report only after evidence confirms all of the following:

1. exact Home Assistant, add-on, and OpenCode versions;
2. provider configuration persisted through a supported mechanism;
3. exact FreeLLMAPI model ID verified;
4. a real request initiated from OpenCode in the Home Assistant environment;
5. evidence that the intended provider/model handled the request;
6. bounded stability observation with no unexpected restarts or retry loops;
7. known rollback and no secret exposure.

## Evidence labels

Use these labels consistently in future reports:

- **VERIFIED:** directly observed with command/UI/API output and timestamp.
- **REPORTED:** supplied by an operator but not independently observed.
- **PROPOSED:** a suggested value or procedure not yet executed.
- **UNKNOWN:** not established by available evidence.
- **OBSOLETE:** once reported but superseded by newer evidence.

## Change log of evidence

| Date | Item | Classification |
|---|---|---|
| 2026-10-27 | Recovery plan and acceptance criteria documented | VERIFIED as documentation; not proof of deployment success |
| 2026-10-27 | `freellmapi_local` configuration insertion and add-on restart reported in the working record | REPORTED |
| 2026-10-27 | Provider API read and real request from Home Assistant OpenCode | UNKNOWN / not demonstrated in the available record |
| 2026-10-30 | Public repository reset to a single root commit before documentation was rebuilt | VERIFIED |

## Source boundary

This repository intentionally contains sanitized, reusable guidance only. Private Home Assistant configuration, credentials, internal state, logs, local addresses, and unrelated project material are not part of the public documentation.
