# Security Policy

## Reporting A Vulnerability

Please report security issues privately by opening a GitHub security advisory or contacting the maintainers through the `pendig` GitHub organization.

Do not disclose provider credentials, webhook tokens, sandbox keys, production keys, or exploit details in public issues.

## Scope

This repository contains an AI Agent skill, not the Rute Bayar application runtime itself. Security-sensitive guidance still matters because the skill can influence how agents handle payment credentials and webhook operations.

In scope:

- Instructions that could cause agents to leak credentials.
- Instructions that weaken webhook verification.
- Unsafe production deployment guidance.
- Malicious or unexpected executable content.

Out of scope:

- Provider API availability.
- Vulnerabilities in third-party payment gateways.
- Issues in the main Rute Bayar repository unless they are caused by this skill's instructions.
