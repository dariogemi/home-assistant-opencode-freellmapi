# Operations and troubleshooting

## Operating rules

- Keep one writer for the add-on and its configuration.
- Make one change per iteration.
- Capture a baseline before a write.
- Use bounded commands and documented timeouts.
- Do not duplicate a request while the first request may still be running.
- Do not add persistent watchers or self-healing loops as a debugging shortcut.
- After each change, validate platform health before attempting another model request.
- Keep rollback material outside the repository.

## Symptom-driven troubleshooting

### OpenCode UI or process is unavailable

1. Check add-on state and recent sanitized logs.
2. Confirm the actual add-on and OpenCode versions.
3. Look for startup/configuration parse errors, port conflicts, or repeated restarts.
4. Do not reinstall, recreate containers, or reset data as a first response.

### Provider is not listed

1. Confirm the configuration was saved through a supported mechanism.
2. Check the exact provider identifier and schema for the installed version.
3. Check for parse/validation errors.
4. Verify persistence through the documented read path.
5. Do not edit internal storage files to force the provider into existence.

### Connection timeout or refused

1. Test from the OpenCode runtime environment.
2. Separate DNS, routing, listener, and firewall hypotheses.
3. Confirm the endpoint and port from the provider's current documentation.
4. Do not expose the endpoint publicly or disable security controls to make a test pass.

### 401 / 403

1. Confirm which authentication mechanism the endpoint documents.
2. Check credential presence and scope without printing the credential.
3. Verify that the credential is valid and has not been revoked.
4. Do not put tokens in command-line arguments, shell history, logs, screenshots, or Git.

### Unknown model or model unavailable

1. Compare the requested model ID with the provider's actual catalog.
2. Distinguish a display label from an API model identifier.
3. Check availability/quota once using a supported provider mechanism.
4. Do not rotate through many models or retry quota/authentication errors automatically.

### API/schema/streaming incompatibility

1. Capture the sanitized status code and error response.
2. Compare the request shape with the provider's documented contract.
3. Verify whether the client requires streaming and whether the provider supports it.
4. Change one verified setting at a time; do not invent undocumented endpoints.

### Add-on crashes or enters a restart loop

1. Stop configuration changes and repeated restarts.
2. Capture the relevant logs and current state.
3. Identify the last change and whether it has a verified rollback.
4. Revert only that change if rollback is safe.
5. If recovery is uncertain, stop and preserve evidence rather than chaining destructive commands.

## Retry policy

- One initial low-cost functional request.
- At most one additional attempt, and only after the first request has definitively ended and the error is plausibly transient.
- Do not automatically retry authentication, authorization, invalid-model, schema, quota, or rate-limit errors.
- Never run load tests or request loops as part of initial integration validation.

## Incident record template

Record the following in a sanitized report:

- timestamp and timezone;
- component and version;
- symptom and exact phase;
- command/action actually executed;
- exit code or HTTP status;
- relevant sanitized log excerpt;
- confirmed facts vs hypotheses;
- change made (if any);
- validation and rollback result;
- final status: PASS, PARTIAL, NO-GO, or UNCERTAIN.

Do not attach raw environment dumps, full configuration exports, databases, internal storage, or logs containing secrets.
