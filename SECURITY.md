# Security policy

## Scope

This repository documents an integration between Home Assistant, OpenCode, and FreeLLMAPI. It is not a place to store deployment secrets or private Home Assistant configuration.

## Never commit

- API keys, access tokens, passwords, cookies, private keys, or authorization headers;
- complete environment dumps or provider configuration containing credentials;
- Home Assistant `.storage`, databases, backups, or internal state;
- raw logs or screenshots that may contain secrets, private addresses, account identifiers, or personal data;
- private network topology or details that are not necessary for a public integration guide;
- files copied from unrelated private projects.

## Safe reporting

When reporting a vulnerability or accidental disclosure:

1. Do not publish the secret or exploit details in an issue.
2. Revoke or rotate exposed credentials at the issuing service.
3. Contact the repository maintainer privately through GitHub.
4. Include the affected path/commit and a concise impact description, without reproducing the sensitive value.

## Operational guidance

- Use a credential store or the supported secret mechanism for the installed version.
- Avoid passing credentials in command-line arguments, shell history, URLs, logs, or screenshots.
- Redact provider responses and logs before sharing.
- Prefer private network paths; do not expose a model endpoint publicly as a troubleshooting shortcut.
- Backups containing secrets must remain outside Git and be access-controlled.

## Scope discipline

Do not change unrelated Home Assistant integrations, add-ons, automations, DNS, firewall, router, or VPN settings as part of this guide unless a separate, reviewed change explicitly includes them.
