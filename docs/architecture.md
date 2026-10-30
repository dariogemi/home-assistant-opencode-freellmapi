# Architecture and component responsibilities

## Purpose

The goal is to let the OpenCode runtime associated with Home Assistant use models served by FreeLLMAPI, while keeping Home Assistant workflows and the model-serving layer independently diagnosable.

This is a logical architecture. The exact integration mechanism must be verified against the installed add-on and integration versions.

## Components

### Home Assistant

Home Assistant owns home-automation state and workflows: automations, scripts, services, events, and user-facing control surfaces.

It should not need to know provider credentials or model-routing internals when those responsibilities can remain inside OpenCode.

### OpenCode runtime

OpenCode is the orchestration and model-client layer. In this deployment it runs in the Home Assistant environment (for example, through an add-on).

Responsibilities include:

- selecting a configured provider and model;
- issuing model requests to the provider endpoint;
- handling provider responses and reporting errors;
- exposing only the supported integration surface needed by Home Assistant.

The OpenCode runtime inside Home Assistant is a distinct target from an OpenCode installation on a workstation. A configuration that works on a workstation is a comparison reference, not proof that the add-on has the same version, environment, network route, or persistence behavior.

### FreeLLMAPI

FreeLLMAPI is the model-serving/routing layer. It receives requests from a compatible client and selects or routes to a model according to its own catalog and policy.

Before treating it as an OpenAI-compatible endpoint, verify the actual API contract, base path, authentication, supported endpoints, request format, streaming behavior, and model identifiers. Do not infer compatibility from a URL suffix or a provider label alone.

## Request path

The intended logical request path is:

1. A user or Home Assistant workflow initiates work through a supported integration boundary.
2. The OpenCode runtime receives the request.
3. OpenCode selects a provider and model configured for FreeLLMAPI.
4. The runtime sends a request over a route reachable from the Home Assistant environment.
5. FreeLLMAPI accepts and routes the request.
6. The model response returns to OpenCode.
7. OpenCode returns the result through the initiating workflow.

A successful test from a workstation proves only the workstation-to-provider path. It does not prove that the Home Assistant add-on can resolve, route to, authenticate with, or use the same endpoint.

## Network boundary

Validate connectivity from the environment where the OpenCode process actually runs. Do not assume that host-level connectivity proves container/add-on connectivity.

A private overlay or VPN may be one possible transport, but it is not an architectural requirement by itself. Use the simplest already-authorized route that is reachable and secure. Do not open inbound ports, weaken a firewall, expose the provider to the public Internet, or change global DNS/routing as an exploratory test.

## Configuration boundary

Keep provider configuration inside the supported OpenCode configuration mechanism for the installed version. Do not edit internal state stores, databases, undocumented files, or Home Assistant storage internals by analogy with another release.

Provider configuration should identify, at minimum, the provider type/protocol, endpoint, authentication mechanism, and model IDs—but the exact field names and persistence mechanism are version-specific.

## Failure domains

| Layer | Typical symptoms | First evidence to collect |
|---|---|---|
| Home Assistant workflow | Request never reaches OpenCode | Automation/service trace and integration logs |
| OpenCode runtime | UI/process unavailable, config parse error | Add-on state, version, sanitized logs |
| Network path | Timeout, DNS or connection failure | Reachability from the runtime environment |
| Authentication | 401/403 or provider authorization error | Credential presence/validity without revealing its value |
| API compatibility | Unsupported route, schema or streaming error | Provider documentation and sanitized response |
| Model routing | Unknown model, unavailable model, quota | Exact model ID and provider-side availability |

Do not collapse these into “the model is down.” Identify the failing boundary first.
